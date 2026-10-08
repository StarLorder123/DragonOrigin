- shell是一个比较常规而且功能比较强大的组件。实现web shell一般来说需要两个进程配合，一个进程拥有v8引擎的环境，一个进程拥有nodejs的环境（或者是类似的环境）。两个进程之间的通信可以使用网络协议，也可以通过进程间的通信通道。
- # 1.  渲染端Shell
- 渲染端需要使用xterm框架，以及对应的插件系列可以搞出一个小黑框用于显示和接收来自服务端的消息。
  
  ```
  import React, { useEffect, useState } from 'react';
  import { Terminal } from 'xterm';
  import { WebLinksAddon } from 'xterm-addon-web-links';
  import { FitAddon } from 'xterm-addon-fit';
  
  import 'xterm/css/xterm.css';
  import TreeFile from './TreeFile';
  
  const FontSize: number = 14;
  const Col = 80;
  
  const WebTerminal = () => {
  const [webTerminal, setWebTerminal] = useState<Terminal | null>(null);
  const [ws, setWs] = useState<WebSocket | null>(null);
  
  useEffect(() => {
    // 新增监听事件
    if (webTerminal && ws) {
      // 监听
      webTerminal.onKey(e => {
        const { key } = e;
        ws.send(key);
      });
  
      // ws监听
      ws.onmessage = e => {
        console.log(e.data);
  
        if (webTerminal) {
          if (typeof e.data === 'string') {
            webTerminal.write(e.data);
          } else {
            console.error('格式错误');
            console.log(e.cancelable);
  
          }
        }
      };
    }
  }, [webTerminal, ws]);
  
  useEffect(() => {
    // 初始化终端
  
    const ele = document.getElementById('terminal');
    if (ele) {
      const height = ele.clientHeight;
      // 初始化
      const terminal = new Terminal({
        cursorStyle: "bar",
        fontSize: 18,
        fontFamily: "'Consolas ligaturized',Consolas, 'Microsoft YaHei','Courier New', monospace",
        disableStdin: false,
        lineHeight: 1.1,
        rightClickSelectsWord: true,
        cursorBlink: true,
        scrollback: 10000,
        tabStopWidth: 8
      });
  
      // 辅助
      const fitAddon = new FitAddon();
      terminal.loadAddon(new WebLinksAddon());
      terminal.loadAddon(fitAddon);
  
      terminal.open(ele);
      terminal.write('Hello from \x1B[1;3;31mxterm.js\x1B[0m $ ');
      fitAddon.fit();
      setWebTerminal(terminal);
    }
  
    // 初始化ws连接
    if (ws) ws.close();
  
    const socket = new WebSocket('ws://127.0.0.1:3001');
    socket.onopen = () => {
      socket.send('connect success');
    };
  
    setWs(socket);
  }, []);
  
  return (
    <div style={{ padding: 40 }}>
      <p>This is a Test</p>
    <div id="terminal" />
      </div>
    );
  };
  
  export default WebTerminal;
  ```
- 这个Render端首先需要链接websocket，然后监听来自websocket的消息显示到渲染端。
- 参考文档：
	- [https://github.com/xtermjs/xterm.js](https://github.com/xtermjs/xterm.js)
- # 2.  服务端Shell
- 服务端就是用于链接远端的服务器的中转。
  
  ```
  import * as ssh2 from 'ssh2';
  import { log } from './log';
  
  // 与Docker链接的一个 SSH 客户端对象
  export class SSH2Client {
  
    private readonly conn: ssh2.Client = new ssh2.Client();
  
    private sftp: ssh2.SFTPWrapper | undefined;
    private stream: ssh2.ClientChannel | undefined;
    private websocket: any;
  
    isReady: boolean = false;
  
    constructor(
        readonly config: ssh2.ConnectConfig,
        websocket: any,
        dataHandler: (data: any) => void,
        errorHandler?: (err: any) => void
    ) {
        this.websocket = websocket;
        this.conn.on('ready', () => {
            this.isReady = true;
            console.log(`SSH Connection Established`);
            log.info(`SSH2Client ${JSON.stringify(config)} Connection Established`);
  
            // 绑定sftp
            this.conn.sftp((err, sftp) => {
                if (err) throw err;
                this.sftp = sftp;
                this.sftp.on('error', (err: any) => {
                    log.error(`SSH2Client stfp ${JSON.stringify(config)} throw error:${err.message}`);
                })
            });
  
            // 绑定shell的stream流
            this.conn.shell((err, stream) => {
                if (err) throw err;
                this.stream = stream;
                this.stream.on('data', dataHandler)
                    .on('error', (err: any) => {
                        log.error(`SSH2Client ${JSON.stringify(config)} throw error:${err.message}`);
                        if (errorHandler) {
                            errorHandler(err);
                        }
                    })
                    .on('close', () => {
                        this.close();
                        log.info(`SSH2Client ${JSON.stringify(config)} has closed`);
                    })
                    .on('exit', (code) => {
                        this.close();
                        log.info(`SSH2Client ${JSON.stringify(config)} has exit. The Code is ${code}`);
                    })
            })
        }).connect(config);
    }
  
    readdirBySftp(path: string) {
        if (this.sftp) {
            this.sftp.readdir(path, (err, list) => {
                if (err) throw err;
                if (this.websocket) {
                    this.websocket.send(JSON.stringify({ type: 'sftp', data: list }));
                }
            });
        }
    }
  
    writeFileBySftp(path: string, data: string | Buffer, options: ssh2.WriteFileOptions) {
        if (this.sftp) {
            this.sftp.writeFile(path, data, options);
        }
    }
  
    writeInStream(data: any) {
        if (this.stream) {
            this.stream.write(data);
        }
    }
  
    // SSH2Client关闭
    close() {
        this.conn.end();
    }
  
    restart() {
        log.info(`SSH2Client ${JSON.stringify(this.config)} has restarted`);
        this.conn.connect(this.config);
    }
  
  }
  ```
  
  ```
  import { log } from "./services/log";
  import { SSH2Client } from "./services/ssh2";
  import * as utf8 from 'utf8';
  
  const express = require('express');
  const app = express();
  require('express-ws')(app);
  
  app.get('/hello/:userID?', (req: any, res: any) => {
    const userId = req.params.userID;
    res.send(`Hello World${userId ?? ''}`);
  });
  
  app.ws('/', (ws: any, req: any) => {
    const ssh2client = new SSH2Client({
        host: '192.168.1.7',
        port: 22,
        username: 'jinhao_li',
        password: '903903'
    }, ws, (data: any) => {
        ws.send(utf8.decode(data.toString('binary')));
    });
    ws.on('message', (data: any) => {
        console.log(data);
        if (typeof data === 'string') {
            const o = JSON.parse(data) as any;
            switch (o.type) {
                case 'sftp':
                    ssh2client.readdirBySftp(o.data);
            }
        }
    });
  });
  
  app.listen(3001);
  
  process.on('uncaughtException', (err) => {
    console.log(`${err.message}`);
    log.error(`${err.message}`);
  })
  ```
- 上述是使用express框架、ssh2框架等实现的服务器端。
- 参考文档：
	- [https://github.com/mscdex/ssh2](https://github.com/mscdex/ssh2)