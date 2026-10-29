# 1.  初识爬虫
- ## 1.1.  初识网络爬虫
- 首先来看一段代码
  
  ```
  from urllib.error import HTTPError
  from urllib.request import urlopen
  from bs4 import BeautifulSoup
  
  try:
    html = urlopen("http://www.pythonscraping.com/pages/page1.html")
  except HTTPError as e:
    print(e)
  if html is None:
    print("URL is not found")
  else:
    print(html.read())
  ```
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607347376045-1c5ccd64-c806-47a2-acd5-d1bd959001e5.png)
- 再来看另外一段代码
  
  ```
  try:
    html = urlopen("http://www.pythonscraping.com/pages/page1.html")
  except HTTPError as e:
    print(e)
  if html is None:
    print("URL is not found")
  else:
    # print(html.read())
    bsObj = BeautifulSoup(html.read())
    print(bsObj.h1)
  ```
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607347416698-34dc4b94-dc56-4724-850a-7ce6c205ed9c.png)
- 这两段代码第一段代码，出现的就是一个网页传输到客户端时最初的样子，就是一堆这样的数据。那是怎么才能变成我们在浏览器看到的那个样子呢？那就是浏览器的功能，浏览器可以对这些数据进行解析，然后呈现出我们的看到的网页页面。然而不同的浏览器支持不同的浏览器内核。这也就是导致在不同浏览器上会出现不同效果的原因。
- 回到原话题。第二段代码出现了报错，通过百度，这个错误可以被忽略，就是在BeautifulSoup这个对象中，没有装载'html.parser'解析器而已。
- 在能正式运行beautifulsoup这个组件之前，只需要在terminal中执行一行这样的命令即可。
	- pip3 install beautifulsoup
- 当我们需要读取哪个标签的时候，直接使用对象.标签名即可。例如：html.h1；这就是读取html标签下的h1这个标签。
- ## 1.2.  复杂的HTML解析
- 当我们面对不同的网页的时候，会发现每个网页都是不同的，而且是复杂的。如果我们不经思考地直接写上下面这样一段代码：
	- bsObj.findAll("table")[4].findAll("tr")[2].find("td").findAll("div")[1].find("a")
- 这样的代码，不仅可读性差，而且一旦网站发生改变之后，这个代码就会立马失效。
- 关于使用find（）和findAll（）函数
	- 具体属性：
		- findAll（tag,attributes,recursive,text,limit,keywords）
		- find(tag,attributes,recursive,text,keywords)
	- recursive是一个布尔变量。如果为true，就可以按照要求去查找标签参数的所有字标签，以及子标签的子标签。如果是false只会查文档的一级标签。
	- limit表示的就是对前x项有兴趣。
	- keywords可以设置用来选择指定属性标签。方法为findAll(id="想要查找的内容")
	- 示例：
	  
	  ```
	  html = urlopen("http://www.pythonscraping.com/pages/warandpeace.html")
	  bsObj = BeautifulSoup(html.read(), 'html.parser')
	  nameList = bsObj.findAll("span", {"class": "green"})
	  for name in nameList:
	   print(name.get_text())
	  ```
	- 以上这个代码就是针对一个网页中的span标签中，class为green的标签进行爬取的。
	- 另外，get_text()这个函数，会把正在处理的HTML文档中所有的标签都清楚，然后返回一个至包含文字的字符串，会把超链接、段落、标签的大段源代码都删除。
- 处理兄弟标签和父标签
	- 主要通过next_siblings、previous_siblings、next_sibling这些api实现的。然后父级标签的查找，主要通过parent和parents对象实现的。
	- 兄弟标签示例：
	  
	  ```
	  html = urlopen("http://www.pythonscraping.com/pages/page3.html")
	  bsObj = BeautifulSoup(html.read(), 'html.parser')
	  
	  for sibling in bsObj.find("table", {"id": "giftList"}).tr.next_siblings:
	   print(sibling)
	  ```
	- 父标签示例：
	  
	  ```
	  html = urlopen("http://www.pythonscraping.com/pages/page3.html")
	  bsObj = BeautifulSoup(html.read(), 'html.parser')
	  
	  print(bsObj.find("img", {"src": "../img/gifts/img2.jpg"}).parent.previous_sibling.get_text())
	  ```
- 正则表达式
	- 正则表达式的规则：
	  
	  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607349086709-f9bacb89-6d3d-4261-9c0e-692df77f461d.png)
	  
	  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607349091959-68684b5e-3371-432a-bcc3-01c65c41c8e5.png)
	- 正则表达式的标准版是基于Perl语法演变而来的。各大主流语言在使用上可能存在差距。
	- 正则表达式的运用：
	  
	  ```
	  html = urlopen("http://www.pythonscraping.com/pages/page3.html")
	  bsObj = BeautifulSoup(html.read(), 'html.parser')
	  images = bsObj.findAll("img", {"src": re.compile("\.\.\/img\/gifts/img.*\.jpg")})
	  for image in images:
	   print(image["src"])
	  ```
	- 代码中的最后一行，其实就是bsObj.findAll函数把所有img标签中的内容，转换成为一个Map的List结构。Map中的key就是属性名，value值就是标签里的内容。
- Lambda表达式
	- Lambda表达式本质上就是一个函数，可以作为其他函数的变量使用；也就是说，一个函数不是定义成f(x,y)，而是定义成f(g(x),y)，或者是f(g(x),h(x))的形式。
	- BeautifulSoup允许把特定的函数类型作为findAll函数的参数，唯一的限制条件是这些函数必须把一个标签作为参数且返回结果是布尔类型。
	- 例如：
		- soup.findAll(lambda tag:len(tag.attrs)==2)
		- 这样的函数就是会返回那些所有标签中含有两个属性的标签。
- # 2.  存储数据
- ## 2.1.  存储媒体文件
- 爬虫爬到想要的东西之后，对于除了文本以外的资源，比如：图片、视频等资源的时候，有两种处理方式：第一种就是只获取URL连接，第二种方式就是直接把源文件下载下来。
- 两种方式的优缺点分析：
	- 只保存连接的优点：
		- 爬虫运行快，耗费流量少。
		- 节省存储空间，只要保存连接就可以。
		- 存储URL的代码更容易写，不需要实现文件下载代码
		- 不下载文件能够降低目标主机的负载量。
	- 只保存连接的缺点：
		- 内嵌在网站中的外站URL被称为盗链，使用盗链可能会导致一系列麻烦。
		- 因为文件在别人的服务器上，应用就需要跟别人的节奏运行了。
		- 盗链是很容易改变的。
		- 现实中的网络浏览器不仅可以请求HTML页面并切换页面，也会下载访问页面上所以的资源，下载就可以让别人觉得这就是人在操作。
- 示例代码：
  
  ```
  import os
  from urllib.request import urlretrieve
  from urllib.request import urlopen
  from bs4 import BeautifulSoup
  
  downloadDirectory = "downloaded"
  baseUrl = "http://pythonscraping.com"
  
  def getAbsoluteURL(baseUrl, source):
    if source.startswith("http://www."):
        url = "http://" + source[11:]
    elif source.startswith("http://"):
        url = source
    elif source.startswith("www."):
        url = source[4:]
        url = "http://" + source
    else:
        url = baseUrl + "/" + source
    if baseUrl not in url:
        return None
    return url
  
  def getDownloadPath(baseUrl, absoluteUrl, downloadDirectory):
    path = absoluteUrl.replace("www.", "")
    path = path.replace(baseUrl, "")
    path = downloadDirectory + path
    directory = os.path.dirname(path)
  
    if not os.path.exists(directory):
        os.makedirs(directory)
  
    return path
  
  html = urlopen("http://pythonscraping.com")
  bsObj = BeautifulSoup(html.read(), 'html.parser')
  downloadList = bsObj.findAll(src=True)
  
  for download in downloadList:
    fileurl = getAbsoluteURL(baseUrl, download["src"])
    if fileurl is not None:
        print(fileurl)
  urlretrieve(fileurl, getDownloadPath(baseUrl, fileurl, downloadDirectory))
  ```
- 这个程序使用了Lambda函数选择首页上所有带src属性的标签。然后对URL连接进行清理和标准化，取得文件的绝对路径。然后就会把文件下载到程序所在的文件夹里。
- ## 2.2.  用csv文件的形式存储
- CSV是存储表格数据的常用文件格式。因为其本身很简洁。
- Python的csv库提供了非常简洁地修改CSV文件，甚至可以从零开始创建一个CSV文件。
- 示例：
  
  ```
  import csv
  import os
  from urllib.request import urlopen
  from bs4 import BeautifulSoup
  
  html = urlopen("https://www.shanghairanking.cn/rankings/bcur/2020")
  bsObj = BeautifulSoup(html.read(), 'html.parser')
  # 主对比表格是当前页面上的第一个表格
  table = bsObj.findAll("table", {"class": "rk-table"})[0]
  rows = table.findAll("tr")
  
  url = os.path.dirname("../files/editors.csv")
  if not os.path.exists(url):
    os.makedirs(url)
  
  csvFile = open("../files/editors.csv", 'wt', newline='', encoding='utf-8')
  writer = csv.writer(csvFile)
  try:
    for row in rows:
        csvRow = []
        for cell in row.findAll(['td', 'th']):
            csvRow.append(cell.get_text())
            writer.writerow(csvRow)
  finally:
    csvFile.close()
  ```
- 以上就是使用代码实现一个对软科大学排名的爬取，然后写入了csv文件当中。
- 效果图：
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607519507476-5adfb4ee-df76-4caa-80df-8badb5d0133f.png)
- ## 2.3.  使用Python连接Mysql数据库
  
  ```
  import pymysql
  
  conn = pymysql.connect(host='cdb-rlerhd0g.cd.tencentcdb.com', port=10078, user='root', password='Just123456', database='mysql')
  cur = conn.cursor()
  cur.execute("USE User")
  
  cur.execute("select * from account")
  print(cur.fetchall())
  cur.close()
  conn.close()
  ```
- 以上就是连接数据库的简单示例。因为我使用的是云数据库，因此主机的host填的就是目标的地址，port就是主机的接口。cur是光标对象。就是小黑框当中的光标。就是通过这个对象操作的。
- 原来的示例中使用的fetchone函数，这只能输出一行。这个示例中使用的事fetchall函数，输出的是所有。
- ## 2.4.  Email的方式
- 与网页通过HTTP协议传输一样，邮件是通过SMTP协议传输的。
- 使用Python发送邮件很容易，只需要几行就可以实现。
- Python包中有两个包可以发送邮件：smtplib和email。
- 示例：
  
  ```
  import smtplib
  import time
  from email.mime.text import MIMEText
  from bs4 import BeautifulSoup
  from urllib.request import urlopen
  
  def sendMail(subject, body):
    msg = MIMEText(body)
    msg['Subject'] = subject
    msg['From'] = "christmas_alerts@pythonscraping.com"
    msg['To'] = "ryan@pythonscraping.com"
    s = smtplib.SMTP('localhost')
    s.send_message(msg=msg)
    s.quit()
  
  bsObj = BeautifulSoup(urlopen("https://isitchristmas.com/"), 'html.parser')
  while(bsObj.find("a", {"id":"answer"}).attrs['title'] == "NO"):
    print("It is not Christmas yet.")
    time.sleep(3600)
  sendMail("It is Christmas!", "According to http://itischristmas.com,it is Christmas!")
  ```
- 以上就是一个简单的检测发送邮件的程序。sendMail函数就是发送邮件的几行代码。
- 而下面这就是对这个网址进行访问，看是否达到能发送邮件的条件，能达到就会发送这个邮件。
- # 3.  文本分析
- ## 3.1.  概括数据
- 在之前已经了解了如何将一个文本内容分解成为n-gram模型，或者说是n个单词长度的词组。从最基本的功能上说，这个集合可以用来确定这段文字中最常用的单词和短语。
- 美国总统文本示例：
  
  ```
  from urllib.request import urlopen
  from bs4 import BeautifulSoup
  import re
  import string
  import operator
  
  def cleanInput(input):
    input = re.sub('\n+', " ", input).lower()
    input = re.sub('\[[0-9]*\]', "", input)
    input = re.sub(' +', " ", input)
    input = bytes(input, "UTF-8")
    input = input.decode("ascii", "ignore")
    cleanInput = []
    input = input.split(' ')
    for item in input:
        item = item.strip(string.punctuation)
        if len(item)>1 or (item.lower() == 'a' or item.lower() == 'i'):
            cleanInput.append(item)
    return cleanInput
  
  def ngrams(input, n):
    input = cleanInput(input)
    output = {}
    for i in range(len(input)-n+1):
        ngramTemp = " ".join(input[i:i+n])
        if ngramTemp not in output:
            output[ngramTemp] = 0
        output[ngramTemp] += 1
    return output
  
  content = str(urlopen("http://pythonscraping.com/files/inaugurationSpeech.txt").read(), 'utf-8')
  ngrams = ngrams(content, 2)
  sortedNGrams = sorted(ngrams.items(), key=operator.itemgetter(1), reverse=True)
  print(sortedNGrams)
  ```
- 这个功能主要就是对词义进行分组，然后会得出一个搭配出现的次数。例如：
- ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607700095827-dd82b9c6-d5a8-409f-ae49-8eece5e421ed.png)
- 经过了分词之后，这就可以看出很多的搭配用法。这属于语法部分，因此按理来说，这个不具有真正的意义，因此可以通过一个最常用5000词的列表来进行过滤，可以过滤掉很多信息。
- ## 3.2.  马尔克夫模型
- 下图就是一个马尔克夫模型的例子。
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607700584118-b30b78ba-f403-496f-87ee-15841ec4bc36.png)
- 由马尔克夫模型演化而来的文字生成器都是基于一种常用于分析大量随机事件的马尔克夫模型，随机事件的特点是一个离散事件之后，另一个离散事件在前一个事件条件下发生的概率。
- 接下来就是一个文本分析和生成的例子：
  
  ```
  from urllib.request import urlopen
  from random import randint
  
  def WordListSum(wordList):
    sum = 0
    for word, value in wordList.items():
        sum += value
    return sum
  
  def retrieveRandomWord(wordList):
    randIndex = randint(1, WordListSum(wordList))
    for word, value in wordList.items():
        randIndex -= value
        if randIndex <= 0:
            return word
  
  def buildWordDict(text):
    # 剔除换行符和引号
    text = text.replace("\n", " ")
    text = text.replace("\"", "")
  
    # 保证每个标点符号都和前面的单词在一起
    # 这样不会被剔除，保留在马尔科夫链中
    punctuation = [',', '.', ';', ':']
    for symbol in punctuation:
        text = text.replace(symbol, " "+symbol+" ")
  
    words = text.split(" ")
    # 过滤空单词
    words = [word for word in words if word != ""]
  
    wordDict = {}
    for i in range(1, len(words)):
        if words[i-1] not in wordDict:
            # 为单词新建一个词典
            wordDict[words[i-1]] = {}
        if words[i] not in wordDict[words[i-1]]:
            wordDict[words[i-1]][words[i]] = 0
        wordDict[words[i-1]][words[i]] = wordDict[words[i-1]][words[i]] + 1
  
    return wordDict
  
  text = str(urlopen("http://pythonscraping.com/files/inaugurationSpeech.txt").read(), 'utf-8')
  wordDict = buildWordDict(text)
  
  # 生成链长为100的马尔科夫链
  length = 100
  chain = ""
  currentWord = "I"
  for i in range(0, length):
    chain += currentWord + " "
    currentWord = retrieveRandomWord(wordDict[currentWord])
  
  print(chain)
  ```
- 结果：
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607700782887-a2097ea0-1b2e-4a99-823c-759eaba66c9e.png)
- 首先是根据文本生成一个马尔克夫链，然后生成一个随机器，用于随机生成文本的。
- ## 3.3.  使用自然语义工具包
- NLTK是一个用于识别和标记英语文本中各个词的词性的包。
- 通过这个包，可以实现很多功能。
- ## 3.4.  Python Request库
- 在一开始的时候就察觉到了Request和urlopen的函数关系。
- 其实request的提交函数和urlopen函数是一样的，方法都是一样的效果，最后的获取也是一样的。
- 但是为了能够进行更加复杂的HTTP请求，cookie等操作。所以需要了Request请求。
  
  ```
  import requests
  params = {'email_addr': 'ryan.e.mitchell@gmail.com'}
  r = requests.post("http://post.oreilly.com/client/o/oreilly/forms/quicksignup.cgi", data=params)
  print(r.text)
  ```
- data的属性就可以用于提交文本、文件、图片等很多形式的内容，实现穿过提交表单进行访问。
- # 4.  读取文档
- ## 4.1.  Python与编码
- 涉及到文件，就需要谈一谈编码的方式。作为程序员都知道ASCII码和UTF-8码，在日常生活中，如果出现了乱码，作为一个中国的程序员，那就是反应，这个码是UTF-8码。但是实际上还有很多种代码，比如UTF-16、UTF-24等等。还有很多的ISO字符集。因此在爬取网站之前，一定要先了解在头部的META标签。
- 例如：
	- <meta charset="utf-8"/>这种标签
- 代码示例：
  
  ```
  from urllib.request import urlopen
  from bs4 import BeautifulSoup
  
  html = urlopen("http://www.pythonscraping.com/pages/warandpeace/chapter1-ru.txt")
  print(str(html.read(), 'utf-8'))
  ```
- 对比效果：
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607575538974-401b75af-e37f-4049-b7b4-62333f565765.png)
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607575544445-85ad724a-bd56-4014-b31b-e355ef99f411.png)
- 第一张图是没转码之前，第二张图是转码之后。
- ## 4.2.  读取CSV文件
- 之前已经会将数据转化成为一个csv文件存储到本地文件了。今天讲的是关于如何将csv文件存储到内存里。
  
  ```
  from urllib.request import urlopen
  from io import StringIO
  import csv
  
  data = urlopen("http://pythonscraping.com/files/MontyPythonAlbums.csv").read().decode("ascii", 'ignore')
  dataFile = StringIO(data)
  dictReader = csv.DictReader(dataFile)
  
  print(dictReader.fieldnames)
  
  for row in dictReader:
    print(row.get("Name"))
  ```
- 这段代码体现的就是对于csv文件的读取。就是将文件中的数据读取到内存中的对象中。
- ## 4.3.  读取PDF文件
- 读取PDF文件，是把PDF读成字符串，然后用StringIO转换成文件对象。
- 示例代码：
  
  ```
  from urllib.request import urlopen
  from pdfminer.pdfinterp import PDFResourceManager, process_pdf
  from pdfminer.converter import TextConverter
  from pdfminer.layout import LAParams
  from io import StringIO
  from io import open
  
  def readPDF(pdfFile):
    rsrcmgr = PDFResourceManager()
    retstr = StringIO()
    laparams = LAParams()
    device = TextConverter(rsrcmgr, retstr, laparams=laparams)
  
    process_pdf(rsrcmgr, device, pdfFile)
    device.close()
  
    content = retstr.getvalue()
    retstr.close()
    return content
  
  pdfFile = urlopen("http://www.pythonscraping.com/pages/warandpeace/chapter1.pdf")
  outputString = readPDF(pdfFile)
  print(outputString)
  pdfFile.close()
  ```
- readPDF函数的最大好处就是，如果PDF文件在电脑里，就可以直接把urlopen返回的对象pdfFile替换成普通的open()文件对象。
- ## 4.4.  对于WORD文件的处理
- word文档普遍对于各类程序不友好，因为很少有软件能够读取word的格式。在2008年以后，微软的Word产品中使用上了docx的文件，采用了类似于xml文件的存储方式。实现了读取。
  
  ```
  import io
  from zipfile import ZipFile
  from urllib.request import urlopen
  from io import BytesIO
  from bs4 import BeautifulSoup
  
  wordFile = urlopen("http://www.pythonscraping.com/pages/AWordDocument.docx").read()
  wordFile = BytesIO(wordFile)
  document = ZipFile(wordFile)
  xml_content = document.read('word/document.xml')
  print(xml_content)
  print("\n")
  
  wordObj = BeautifulSoup(xml_content.decode('utf-8'), 'html.parser')
  textStrings = wordObj.findAll("w:t")
  for textElem in textStrings:
    closeTag = ""
    try:
        style = textElem.parent.previousSibling.find("w:pstyle")
        if style is not None and style["w:val"] == "Title":
            print("<h1>")
            closeTag = "</h1>"
    except AttributeError:
        pass
    print(textElem.text)
    print(closeTag)
  ```
- ## 4.5.  数据清洗
- 在语言学中有一个模型叫n-gram，表示文字或语言中的n个连续的单词组成的序列。在进行自然语言分析时，使用n-gram或者寻找常用词组，可以很容易地把一句话分解成若干个文字片段。
- 对于已经分片之后的字段，我们还是可以发现对于文本中，还是会存在一些脏数据、或者是不规范的文字，需要被替换或改变。
- 每个人都会遇到一些样式设计不够人性化的网页。这时候就是考验数据标准化的技术的时候。
- # 5.  采集过程
- ## 5.1.  遍历一层的网络
- 遍历一层网站的意思就是，不使用递归的情况下，对网站进行单层的遍历。遍历的速度就是会很快。
  
  ```
  from urllib.request import urlopen
  from bs4 import BeautifulSoup
  import time
  import re
  
  pages = set()
  def getLinks():
    global pages
    html = urlopen("https://www.shanghairanking.cn/rankings/bcur/2020")
    bsObj = BeautifulSoup(html.read(), 'html.parser')
    # 找到与这个匹配规则相同的文本连接
    for link in bsObj.findAll("a", href=re.compile("^(/institution/)")):
        if 'href' in link.attrs:
            if link.attrs['href'] not in pages:
                newPage = link.attrs['href']
                print("New page is "+newPage)
                pages.add(newPage)
  
  startTime = time.time()
  getLinks()
  endTime = time.time()
  print("运行时间：" + (endTime - startTime))
  ```
- 运行结果：
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607409707816-bd635dac-a9b4-4f4b-8d7c-ad4ee92bf695.png)
- 可以发现，这一段代码就花了1秒钟时间就完成了执行，这就是一层遍历的速度。
- 这一段代码中，getLinks函数就执行了对[https://www.shanghairanking.cn/rankings/bcur/2020](https://www.shanghairanking.cn/rankings/bcur/2020)这个网址进行了单层次的扫描。对html根标签下的a标签进行扫描，符合../institution/..这个模式的网页就会被筛选出来。
- 然后通过set数据结构下的特性。set是一个集合，里面的子集是不会产生重复的。
- ## 5.2.  深度遍历整个网站
- 深度遍历的说法就是一旦遇到一个连接，即网站内的内链。就会深入地再进去遍历，直到网页内没有内链。然后通过递归的形式完成深度的遍历。
  
  ```
  from urllib.request import urlopen
  from bs4 import BeautifulSoup
  import datetime
  import random
  import re
  
  random.seed(datetime.datetime.now())
  
  def getLinks():
    html = urlopen("https://www.shanghairanking.cn/rankings/bcur/2020")
    bsObj = BeautifulSoup(html.read(), 'html.parser')
    # https://www.shanghairanking.cn/institution/tsinghua-university
    return bsObj.find("table", {"class": "rk-table"}).findAll("a", href=re.compile("^(/institution/)((?!:).)*$"))
  
  links = getLinks()
  while len(links) > 0:
    newArticle = links[random.randint(0, len(links)-1)].attrs["href"]
    print(newArticle)
    links = getLinks()
  ```
- 可以发现，上一个代码没有使用递归，但是这一个使用了递归之后，就可以看出来，这个的运行效率明显小于第一个。
- ## 5.3.  通过互联网采集信息
- 网络爬虫位于许多新式的网络技术领域彼此交叉的中心地带。为了能获得更多的信息，可以通过互联网的形式来实现对于数据的采集。就是一旦发现一个外部链接之后，不是就采集地址，就直接进入进行新一轮的采集信息。
- 为了能够采集更加有效的信息，在查找的方式上要极具灵活性，这就可以通过几个灵活的Python函数组合来实现不同类型的网络爬虫。
  
  ```
  from urllib.request import urlopen
  from bs4 import BeautifulSoup
  import re
  import datetime
  import random
  
  pages = set()
  random.seed(datetime.datetime.now())
  
  # 获取页面内所有内链的列表
  def getInternalLinks(bsObj, includeUrl):
    internalLinks = []
    # 找出所有以“/”开头的连接
    for link in bsObj.findAll("a", href=re.compile("^(/|.*"+includeUrl+")")):
        if link.attrs['href'] is not None:
            if link.attrs['href'] not in internalLinks:
                internalLinks.append(link.attrs['href'])
    return internalLinks
  
  # 获取页面所有外链的列表
  def getExternalLinks(bsObj, excludeUrl):
    externalLinks = []
    # 找出所有以“http”或“www”开头且不包含当前url的连接
    for link in bsObj.findAll("a", href=re.compile("^(http|www)((?!"+excludeUrl+").)*$")):
        if link.attrs['href'] is not None:
            if link.attrs['href'] not in externalLinks:
                externalLinks.append(link.attrs['href'])
    return externalLinks
  
  # 地址替换
  def splitAddress(address):
    addressParts = address.replace("http://", "").split("/")
    return addressParts
  
  # 随机外链连接
  def getRandomExternalLink(startingPage):
    html = urlopen(startingPage)
    bsObj = BeautifulSoup(html.read(), 'html.parser')
    externalLinks = getExternalLinks(bsObj, splitAddress(startingPage)[0])
    if len(externalLinks) == 0:
        internalLinks = getInternalLinks(startingPage)
        return getRandomExternalLink(externalLinks)
    else:
        return externalLinks[random.randint(0, len(externalLinks)-1)]
  
  def followExternalOnly(startingSite):
    externalLink = getRandomExternalLink(startingSite)
    print("随机外链是："+externalLink)
    followExternalOnly(externalLink)
  
  followExternalOnly("http://oreilly.com")
  ```
- ## 5.4.  API的认知
- API可以是一个环境内的一些包，然后通过导入等手段就可以直接实现操作；也可以是一个网络的接口，通过访问并传递一定量的数据，就可以获得相应的以xml或者json格式返回的数据流。
- 解析json数据
  
  ```
  def getCountry(ipAddress):
    response = urlopen("http://freegeoip.net/json/"+ipAddress).read().decode('utf-8')
    print(response)
    responseJson = json.loads(response)
    return responseJson.get("country_code")
  
  print(getCountry("50.78.253.58"))
  ```
- # 6.  避开陷阱
- ## 6.1.  避开采集陷阱
- 为了能够合法采集信息，我们需要首先知道每个网页的robot.txt，了解网站那些东西允许被采集，那些不允许被采集。
- 网站防采集的前提就是正确地区分人类访问用户和网络机器人。虽然网站可以使用很多识别技术（比如验证码）来防止爬虫，但是爬虫还是有很多的方法来实现更加像人类用户。
- 第一个方法就是添加请求头：
  
  ```
  import requests
  from bs4 import BeautifulSoup
  
  session = requests.Session()
  headers ={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9"
  }
  url = "https://www.whatismybrowser.com/developers/what-http-headers-is-my-browser-sending"
  req = session.get(url, headers=headers)
  
  bsObj = BeautifulSoup(req.text, 'html.parser')
  print(bsObj.find("table", {"class": "table-striped"}).get_text)
  ```
- 第一个方法就是使用request模块，在提交请求的时候，就将UserAgent的相关的信息提交上去，伪装成一个人类用户。
- 第二个方法就是处理Cookie。
- 第二个方法就是针对cookie这一双刃剑进行处理的。cookie有时候会被用来跟踪用户的访问过程，如果发现了爬虫的异常行为，就会中断你的访问。
  
  ```
  from selenium import webdriver
  
  driver = webdriver.Chrome()
  driver.get("http://pythonscraping.com")
  driver.implicitly_wait(1)
  print(driver.get_cookies())
  
  savedCookies = driver.get_cookies()
  
  driver2 = webdriver.Chrome()
  driver2.get("http://pythonscraping.com")
  driver2.delete_all_cookies()
  for cookie in savedCookies:
    driver2.add_cookie(cookie)
  
  driver2.get("http://pythonscraping.com")
  driver.implicitly_wait(1)
  print(driver2.get_cookies())
  ```
- 通过这个上面的示例可以看出来，两个webdriver是同一个，因为两个的webdriver所含的cookie顺序虽然不一样，但是内容却是一样的。
- ## 6.2.  表单安全措施
- 最主要的就是隐含输入字段值。
- 第一种就是在表单页面上的一个字段可以用服务器生成的随机变量表示。如果提交的时候，这个字段没有出现，那么而是由机器人直接提交的，那就可以认为这个提交不是原始页面上提交的。这个方式一般可以通过扫描原来提交网页上的生成随机变量，然后再提交。
- 第二种就是蜜罐陷阱。就是在网页中隐藏一部分提交的信息。作为爬虫，他是可以检测到，然后填写之后进行提交。但是作为人为的提交表单是看不到这个隐藏信息的。一旦提交，就会触发封杀IP地址。
  
  ```
  from selenium import webdriver
  from selenium.webdriver.remote.webelement import WebElement
  
  driver = webdriver.Chrome()
  driver.get("http://pythonscraping.com/pages/itsatrap.html")
  links = driver.find_elements_by_tag_name("a")
  
  for link in links:
    if not link.is_displayed():
        print("The link"+link.get_attribute("href")+" is a trap.")
  
  fields = driver.find_elements_by_tag_name("input")
  for field in fields:
    if not field.is_displayed():
        print("Do not change value of " + field.get_attribute("name"))
  ```
- 以上示例的代码，就是查找相应的隐藏字段和连接，确保提交之前，确认这些没有被提交，否则就会被封杀。
- ## 6.3.  问题检查表
- 网页空白：可能网站创建页面的JavaScript执行出现问题。
- 发送POST请求的时候，在发之前确认一下所有字段是否都填好，确保格式正确。
- 已经登录的网站无法保证登录状态，或出现了其他的登录状态异常，请检查cookie。
- 在客户端遇到HTTP 403错误。此时有可能网站已经将你这个IP认为是机器人，禁止访问了。要么等待从黑名单移除，要么就换一个IP。
- 在进行爬虫的时候，一定要注意不能快速采集、修改请求头，这样能保证你的爬虫不会很快就被封杀。
- 如果遇到用了一大堆复杂的手段才接入网站，不如联系一下网管吧。
- ## 6.4.  Python+Selenium进行自动化测试
- 原先Selenium框架是用于进行自动化测试的。后来才发现了能用来做爬虫的特质。
- Python的单元测试模块unittest，所有标准版Python安装之后都有。导入模块之后，继承unittest.TestCase类，就可以实现以下的功能：
	- 为每个单元测试的开始和结束提供setUp和tearDown函数
	- 提供不同类型的断言语句让测试成功或失败
	- 把所有以test_开头的函数当做单元测试运行，忽略不带test_的函数。
- Python的但愿测试语法严谨冗长，更适合为大多数大型项目写测试；Selenium的测试方法灵活且功能强大，可以用于测试网站。两者搭配会是一个很好的选择。
  
  ```
  from selenium import webdriver
  from selenium.webdriver.remote.webelement import WebElement
  from selenium.webdriver import ActionChains
  import unittest
  
  class TestAddition(unittest.TestCase):
    driver = None
    def setUp(self):
        global driver
        driver = webdriver.Chrome()
        url = "http://pythonscraping.com/pages/javascript/draggableDemo.html"
        driver.get(url)
  
    def tearDown(self):
        print("Tearing down the test")
  
    def test_drag(self):
        global driver
        element = driver.find_element_by_id("draggable")
        target = driver.find_element_by_id("div2")
        actions =ActionChains(driver)
        actions.drag_and_drop(element, target).perform()
  
        self.assertEqual("You are definitely not a bot", driver.find_element_by_id("message").text)
  
  if __name__ == "__main__":
    unittest.main()
  ```
- 以上就是两个东西组合之后，能实现的巨大功能。
- # 7.  Selenium
- ## 7.1.  初识Selenium
- Selenium是一个强大的网络数据采集工具，其最初的目标是为了网站自动化测试而开发的。近几年，被广泛应用于获取精准的网站快照，因为可以直接运行在浏览器上。
- Selenium可以让浏览器自动加载页面，获取需要的数据，甚至是截屏。
- Selenium自己不带浏览器，需要和第三方浏览器挂钩。
- PhantomJS这个无头的浏览器已经被彻底弃用了，因此，需要更换另外的浏览器。我主要使用的就是Chrome浏览器。
- ## 7.2.  实现Selenium+Chrome联动
- 在命令行实现了pip3 install Selenium之后，还需要进行一系列的配置。
- 要实现对Chrome的自动操作，要使用到ChromeDriver。直接上镜像网站，下载与Chrome浏览器版本相同的ChromeDrvier。自Chrome71版本之后，Chrome和ChromeDrvier已经实现了版本号相同。
- 下载之后，解压出来，放置在Chrome浏览器的根目录位置。我的位置为C:\Program Files\Google\Chrome\Application
- 要实现能操作Chrome，还需要将这个驱动放置在python的环境根目录D:\Python37下。
- 然后在环境变量中配置Path变量。
- 接下来就可以开始调用Selenium进行操作了。
- 示例：
  
  ```
  from selenium import webdriver
  import time
  
  driver = webdriver.Chrome()
  driver.get("http://pythonscraping.com/pages/javascript/ajaxDemo.html")
  time.sleep(3)
  print(driver.find_element_by_id('content').text)
  driver.close()
  ```
- 效果：
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607749066890-2455d73c-ab33-41c1-b411-ed4b2802e794.png)
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607749071013-499b7855-7742-4e7e-b5bc-236b5d1f2d9d.png)
- ## 7.3.  Selenium对页面的检测
- 因为我们向网站提出请求，然后网站收到请求之后，进行一定的处理，然后返回网页，需要一定的时间。为了更加安全有效地进行下面的操作。我们可以让Selenium不断检测某个元素是否存在，以此来确定页面是否已经完全加载，如果页面加载成功就执行后面的程序。
- 示例：
  
  ```
  from selenium import webdriver
  from selenium.webdriver.common.by import By
  from selenium.webdriver.support.ui import WebDriverWait
  from selenium.webdriver.support import expected_conditions as EC
  import time
  
  driver = webdriver.Chrome()
  driver.get("http://pythonscraping.com/pages/javascript/ajaxDemo.html")
  try:
    element = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.ID, "loadedButton"))
    )
  finally:
    print(driver.find_element_by_id('content').text)
    driver.close()
  ```
- 程序里新导入了一些新的模块，需要注意的就是WebDriverWait和expected_conditions。这两个模块组合起来就构成了Selenium的隐式等待。隐式等待是等DOM中某个状态发生后在继续运行代码，而显式等待明确设置了等待时间。
- 这里面还使用了定位器——By。定位器是一种抽象的查询语言，用By对象表示，可以用于不同的场合，包括创建选择器。
- 以下是定位器的一些使用：
	- ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607749596119-2a71740e-6a23-42d3-875e-16a73ae4d810.png)
- ## 7.4.  对于重定向的检测
- 对于重定向，我们可以在加载前就盯住某一个元素，重复调用这个元素，直到Selenium抛出一个异常，也就是说，这个元素不在DOM中，就说明这个网页实现了跳转。
  
  ```
  from selenium import webdriver
  import time
  from selenium.webdriver.remote.webelement import WebElement
  from selenium.common.exceptions import StaleElementReferenceException
  
  def waitForLoad(driver):
    elem = driver.find_element_by_tag_name("html")
    count = 0
    while True:
        count += 1
        if count > 20:
            print("Timing out after 10 seconds and returning")
            return
        time.sleep(5)
        try:
            elem == driver.find_element_by_tag_name("html")
        except StaleElementReferenceException:
            return
  
  driver = webdriver.Chrome()
  driver.get("http://pythonscraping.com/pages/javascript/redirectDemo1.html")
  waitForLoad(driver)
  print(driver.page_source)
  ```
- 这个程序每半分钟检查一次网页，看看标签还在不在。检查的时间间隔和时限都可以根据实际情况随意调整。
- ## 7.5.  Tesseract+Selenium实现网页的图片信息抓取
- 单纯使用Tesseract读取硬盘上的图片信息，效果并不理想，但是如果能配合Python的爬虫，就可以形成一个强大的功能。
- 示例就首先导航到了一个大号印刷版的网站，然后收集URL连接图片，最后进行识别等工作。
  
  ```
  import time
  from urllib.request import urlretrieve
  import subprocess
  from selenium import webdriver
  
  driver = webdriver.Chrome()
  driver.get("https://www.amazon.com/War-Peace-Leo-Nikolayevich-Tolstoy/dp/1427030200")
  time.sleep(2)
  
  # 单击图书预览按钮
  driver.find_element_by_id("sitbLogoImg").click()
  imageList = set()
  
  # 等待页面加载完成
  time.sleep(5)
  # 当向右箭头可以点击时，开始翻页
  while "pointer" in driver.find_element_by_id("sitbReaderRightPageTurner").get_attribute("style"):
    driver.find_element_by_id("sitbReaderRightPageTurner").click()
    time.sleep(2)
    # 获取已经加载的新页面（一次可以加载多个月面，但是重复的页面不能加载到集合中）
    pages = driver.find_elements_by_xpath("//div[@class='pageImage']/div/img")
    for page in pages:
        image = page.get_attribute("src")
        imageList.add(image)
  
  driver.quit()
  print(len(imageList))
  
  # Tesseract处理我们收集的图片URL连接
  for image in sorted(imageList):
    print("Processing....")
    urlretrieve(image, "page.jpg")
    p = subprocess.Popen(["tesseract", "page.jpg", "page"], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    p.wait()
    f = open("page.txt", "r")
    print(f.read())
  ```
- # 8.  爬取实践
- ## 8.1.  爬取知乎
- 今天开始看另外一本网络爬虫的书。发现有好多内容重复了，所以跳着看了看。然后发现了一个书后的实战示例。
- 给书后一个评价：这什么玩意！最后还是用了我自己的方法来实现对于这个知乎直播部分的爬取。
- 今天顺着昨天的思路，首先看了看这个网页。发现：这个网页挺好的，返回回来的时候，直接就是完整的网页，这可算是省了很多的力气。
  
  ```
  https://api.zhihu.com/lives/homefeed?includes=live
  ```
- 所有关于直播板块的数据都存放在这个url中，直接提出之后，就可以进行访问。
- 但是，尝试了一个之后，又发现问题了
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1608048517958-3dafc10a-b1f3-4f82-98d7-7797650f98c6.png)
- 这样的一个页面，完整显示的数据，就直到讲座作者就完事了！然后那些个评论数据又是放在一个url里。好无语啊！
- 后来终于在三级页面里找到了请求的这个网址，而且这个网址用于请求的属性，并不算难。本以为其中的sku_id属性用于表示的是原来的用户ID。其实这个评论的属性另有文章。
- 太生气了，搞这么复杂。通过比对，发现了这个不同，我想应该存在于网页中的某个位置，因为在请求中，没有什么额外的请求。果然，在一堆的textarea中找到了这个属性。这个json复杂程度令人发指！
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1608049130260-09219340-7210-4e3d-b637-5759770b1aab.png)
- 最后还是提取出来，然后做了改造。
- 示例代码
  
  ```
  import csv
  import json
  import os
  
  import requests
  from bs4 import BeautifulSoup
  
  def Collect():
    with open("../files/知乎.csv", "r", encoding='UTF-8') as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            # 获取每个评论中的id号
            id = row["id"]
            # 这是一个直播模块的网址，用于参考。
            # https: // www.zhihu.com / lives / 919165878388535296
            tempUrl = "https://www.zhihu.com/lives/" + id
            print(tempUrl)
            html = requests.get(url=tempUrl, headers=header)
            bsObj = BeautifulSoup(html.content, 'lxml', from_encoding='utf-8')
            # html = urlopen(tempUrl)
            # bsObj = BeautifulSoup(html, 'html.parser')
            # 找到这个论坛的创始人的标签
            Writer = bsObj.find("div", {"class": "UserCell-name-hzEyq"}).get_text()
            print("Writer:", Writer, '\n')
            # 查找到含有评论网页json的sku_id号
            TextArea = bsObj.find("textarea", {"id": "resolved"}).get_text()
            Json = json.loads(TextArea)
            sku_id = Json["appContext"]["__connectedAutoFetch"]["liveDetailsNew"]["data"]["skuGrading"]["sku_id"]
            print(sku_id)
            # 如果不存在这个文件，就要创建一个项目
            tempFileName = os.path.dirname("../files/" + id + ".csv")
            if not os.path.exists(tempFileName):
                os.makedirs(tempFileName)
            tempCsvFile = open("../files/" + id + ".csv", 'wt', newline='', encoding='utf-8')
            writer = csv.DictWriter(tempCsvFile, fieldnames=['author', 'comment'])
  
            # https://api.zhihu.com/darwin/hot_reviews?sku_id=969583563645173760&limit=10&offset=0
            commentUrl = "https://api.zhihu.com/darwin/hot_reviews?sku_id=" + sku_id + "&limit=10&offset=0"
            comment_res = requests.get(url=commentUrl, headers=header)
            for item in comment_res.json()["data"]:
                author = item["author"]["name"]
                comment = item["content"]
                print(author, ":", comment)
                writer.writerow({"author": author, "comment": comment})
            tempCsvFile.close()
  
  if __name__ == '__main__':
    # 将知乎直播模块的url网址存放进url变量中，将请求头设置完成
    url = "https://api.zhihu.com/lives/homefeed?includes=live"
    header = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36"
    }
    # 发送请求
    response = requests.get(url=url, headers=header)
  
    print(response.text)
  
    # 设定一个用于存储讨论课题和id号的文件
    fileUrl = os.path.dirname("../files/知乎.csv")
    # 如果不存在这个文件，就要创建一个项目
    if not os.path.exists(fileUrl):
        os.makedirs(fileUrl)
    csvFile = open("../files/知乎.csv", 'wt', newline='', encoding='utf-8')
    writer = csv.DictWriter(csvFile, fieldnames=['title', 'id'])
    # 写入列标题，即DictWriter构造方法的fieldnames参数
    writer.writeheader()
  
    for item in response.json()["data"]:
        id = item["live"]["id"]
        title = item["live"]["subject"]
        print(id, ":" + title)
        writer.writerow({'title': title, 'id': id})
    csvFile.close()
    Collect()
  ```
- 这里面，提供了两种BeautifulSoup解析的方式。调查了一下，主要是因为request.get返回的对象，和urlopen返回的对象不一样。request要使用lxml的解析器，而urlopen只需要使用html的解析器即可。
- 然后今天也对csv文件进行了一个进一步的拓展学习。可以发现，之前就是直接写，今天进行了一个改进，就是对格式进行了调整。
- 不足之处
	- 主要就是最近读了代码整洁之道。我觉得这一段代码，就很不美。但是也是爬虫初学者。我之后会注意这方面的问题。尽量将方法普遍化。
- ## 8.2.  爬取美团
- 学了爬虫一周，今天就想实战一下，爬取美团的评论。
- 一开始觉得爬取美团应该很容易，稍微研究一下美团的网站结构就行。但是还是太年轻，毕竟也是一个体量比较大的公司，怎么可能随意让我爬取呢？
- 废话不多说。
- 首先是，我选择一个商家的页面，想要一展身手，爬取数据。
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607960222669-a1feda00-1d17-43f3-bffc-369088f1ae9e.png)
- 为这个页面，我设计了一段爬取的代码。
  
  ```
  import csv
  import os
  import time
  
  import requests
  from bs4 import BeautifulSoup
  from selenium import webdriver
  
  # 用于存放和创建爬虫爬的文件信息的csv文件
  def WriteCSV(filename, rows):
    url = os.path.dirname("../files/"+filename+".csv")
    if not os.path.exists(url):
        os.makedirs(url)
  
    csvFile = open("../files/"+filename+".csv", 'wt', newline='', encoding='utf-8')
    writer = csv.writer(csvFile)
    try:
        for row in rows:
            csvRow = []
            name = row.find("div", {"class": "name"})
            desc = row.find("div", {"class": "desc"})
            print(name.get_text()+":"+desc.get_text()+"\n")
            csvRow.append(name)
            csvRow.append(desc)
            writer.writerow(csvRow)
  
    finally:
        csvFile.close()
  
  # 设置存储的文件名
  filename = "meitu"
  # 设置请求头和相应的参数
  header ={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36",
  }
  url = "https://www.meituan.com/meishi/1166119325/"
  # 请求网页并设置相关的编码
  res = requests.get(url=url, headers=header)
  res.encoding = res.apparent_encoding
  time.sleep(.5)
  # res返回的是一个是否发送成功的网页代码，res.text才是真正的网页信息
  # print(res.text)
  bsObj = BeautifulSoup(res.text, 'html.parser')
  print(bsObj)
  infos = bsObj.find_all(attrs={'class': 'info'})
  for info in infos:
    print(info.get_text())
  
  WriteCSV(filename, infos)
  print("完成读写")
  ```
- 以上代码就是，一个为其设计的逻辑上的代码。事实证明，这个代码可以跑，但是没有任何效果。为什么呢？
- 通过多次执行之后，发现了：当我们一开始请求之后，如果就是单纯的打印出来，而不是交由浏览器进行ajax解析的话，其实就是一个页面下，有很多很多的script。也就是说，我们向美团的服务器请求之后，返回的不是一个直接就看到的页面，而是一大段的javascript代码，由这些代码，再向其他地方进行请求相应的数据。
- 尴尬！通过一系列的csdn，发现了这样的一个事实：虽然碰到了钉子，但是发现。美团的数据全都存在于XHR包中。
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1607960498132-5c04c8d2-6d13-42d1-a638-896c6798a7dc.png)
- 上图的右侧可以看到这一事实。其实真实的流程应该是：向服务器请求→返回javascript→javascript代码执行ajax向其他服务器进行请求→返回json文件及其他文件→看到我们看到的页面。
- 因此，爬取一个商家的评论其实就等于请求一个json文件，然后解析。
- 当然我并不满足这样一个情况，我就在思考，如果我想自动爬取多个商家呢？于是我就向上一级网页探索。可以发现，二级网页向三级网页（也就是商家网页跳转的时候）在二级页面上会带有poiId这样一个属性，这个属性就是精准获取数据的标识符。
- 但是在爬二级网页的时候，就发现一个问题，那就是向二级请求的时候，会附带一个_token属性，这个东西就像一个密匙一样，每一次跳转都不一样，因此无法像人一样进行手动跳转了。因此就放弃了这样的一个想法。
- 转念一想，我可以通过Selenium+Chrome的自动化方式来进行跳转啊。于是我又写了一个小代码尝试了一下。
  
  ```
  import requests
  url = "https://www.meituan.com/meishi/api/poi/getMerchantComment?uuid=3bfecc6d-888e-41e2-9082-d6f6ac2e8721&platform=1&partner=126&originUrl=https%3A%2F%2Fwww.meituan.com%2Fmeishi%2F193383554%2F&riskLevel=1&optimusCode=10&id=193383554&userId=&offset=10&pageSize=10&sortType=1"
  
  headers_mt = {
    "User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36"
  }
  reponse = requests.get(url=url, headers=headers_mt)
  print(type(reponse))
  
  for item in reponse.json()["data"]["comments"]:
        name = item["userName"]
        user_id = item["userId"]
        user_url = item["userUrl"]
        comment = item["comment"]
        print(name,user_id,user_url,comment)
        print("~"*60)
  ```
- 这一次就直接进的是美团的主页，可以发现，进都进不去。直接跳403错误。我以为是我的IP地址被封了。我自己手动点击，却是可以的。可见不太可能是封杀的问题。因为美团会根据地域进行推荐，我想在进入美团主页的时候，肯定提供了类似地址的属性，但是我不知道怎么提交了。这样一来，爬虫机器人一眼被识破，因此会被阻止访问。
- 因此，我最后实现了一个，将二级网页的json文件复制下来之后，通过读取文件的形式，对三级商家页面实现遍历，并将数据存入csv文件中。
  
  ```
  import csv
  import json
  import os
  import time
  
  import requests
  import subprocess
  from selenium import webdriver
  from bs4 import BeautifulSoup
  
  # 设置请求头和相应的参数
  header ={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36",
  }
  # 通过手动的方式连接到相应的数据。这个文件是一个json文件。
  # 里面存储的就是关于这一商业区内，这一页内的数据。
  # 以江苏大学商圈内的店铺为例，这里面共有79家商店，每页一共存有14家店铺
  # url = "https://zj.meituan.com/meishi/"
  
  # 请求网页并设置相关的编码
  # res = requests.get(url=url, headers=header)
  
  with open("../files/meitu_json.txt", encoding='utf-8') as file:
    res = file.read()
    print(res.strip())
  
  j = json.loads(res)
  print(type(j))
  
  url = os.path.dirname("../files/江苏大学.csv")
  if not os.path.exists(url):
    os.makedirs(url)
  
  csvFile = open("../files/江苏大学.csv", 'wt', newline='', encoding='utf-8')
  writer = csv.writer(csvFile)
  
  for item in j["data"]["poiInfos"]:
    # 首先获取的是这一页上商家的poiId号
    poiId = item["poiId"]
    title = item["title"]
    print((str)(poiId)+":"+title+"\n")
    # https://www.meituan.com/meishi/api/poi/getMerchantComment?uuid=3bfecc6d-888e-41e2-9082-d6f6ac2e8721&platform=1&partner=126&originUrl=https%3A%2F%2Fwww.meituan.com%2Fmeishi%2F5085299%2F&riskLevel=1&optimusCode=10&id=5085299&userId=&offset=0&pageSize=10&sortType=1
    comment_url = "https://www.meituan.com/meishi/api/poi/getMerchantComment?" \
          "uuid=3bfecc6d-888e-41e2-9082-d6f6ac2e8721&" \
          "platform=1&partner=126&" \
          "originUrl=https%3A%2F%2Fwww.meituan.com%2Fmeishi%2F"+(str)(poiId)+"%2F&riskLevel=1&" \
                                                                      "optimusCode=10&id="+(str)(poiId)+"&userId=&offset=0&pageSize=10&sortType=1"
    # 请求网页并设置相关的编码
    commentsRes = requests.get(url=comment_url, headers=header)
    # 查询已经返回的信息并对其中的信息进行加工处理
    for comment_item in commentsRes.json()["data"]["comments"]:
        csvRow = []
        userName = comment_item["userName"]
        comment = comment_item["comment"]
        print(userName, ":" + comment)
        csvRow.append(userName)
        csvRow.append(comment)
        writer.writerow(csvRow)
    time.sleep(.5)
  csvFile.close()
  ```
-