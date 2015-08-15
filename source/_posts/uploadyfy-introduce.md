title: 文件上传插件uploadify的简单介绍
id: 29
categories:
  - javascript
  - php
  - 前端开发
date: 2014-10-01 00:00:00
tags:
---

熟悉使用php的人一定会对php文件上传不陌生，多文件上传以及重命名等都是文件上传的重要应用，我在这里介绍一个ajax+jQuery的插件，这个插件比较好的地方就是前端的效果比较炫酷，使用比较简单，大大简化了代码复杂度。uploadify.swf是插件的核心部分，（下载地址:http://www.uploadify.com/download/）前端的效果主要由jquery.uploadify.js实现，所以这两个文件要包含进前端的源代码。我在这里主要是想介绍前端的接口函数以及php后台里面的文件重命名的技巧。
前端比较简单，主要是文件包含以及文件上传表单和接口函数。需要包含这几个js文件

```javascript
<script src="jquery.min.js" type="text/javascript"></script>
<script src="jquery.uploadify.min.js" type="text/javascript"></script>
```

还有就是接口函数:

```javascript
<script type="text/javascript">

$(function() {
$('#file_upload').uploadify({
'swf' : 'uploadify.swf',
'uploader' : 'uploadify.php'
});
});
</script>

```

接口函数里面的uploadify()是属于插件uploadify.swf的函数，所以必须先把文件包含进去。
前端body部分的代码如下：

```html
<form>
<div id="queue"></div>
<input id="file_upload" name="file_upload" type="file" multiple="true">
</form>
```
后台部分的php代码比较简单，比较关键的部分就是重命名，根据接口，后台处理函数命名为uploadify.php。由于多文件上传可以在一秒钟之内完成多个文件上传，所以把时间戳作为唯一命名的做法需要注意的地方就是不能把time函数作为字符串，这样会导致字符串相同，如此便没办法上传多个文件，我在这里的解决办法就是使用microtime()生成微秒级的时间，再转化为字符串，这样每个文件的命名就会唯一了。主要代码如下：

```php
// Define a destination
$targetFolder = '/uploads'; // Relative to the root

/*$verifyToken = md5('unique_salt' . $_POST['timestamp']);*/

if (!empty($_FILES) &amp;&amp; $_POST['token'] == $verifyToken) {
$tempFile = $_FILES['Filedata']['tmp_name'];
$targetPath = $_SERVER['DOCUMENT_ROOT'] . $targetFolder;

/*文件以时间戳重命名，这6行代码为个人添加，可以删除，删除之后保持文件原名*/
$date = microtime();//取得当前时间
$nodot = explode(".",$date);//切割掉 microtime()函数返回数据格式 0.25139300 1138197510中的点
$namestr = str_replace(' ','',$nodot[1]);//去除空格
$uptype = explode(".", $_FILES["Filedata"]["name"]);//分割留下文件类型
$newname = $namestr.".".$uptype[1];
$_FILES["Filedata"]["name"] = $newname;//重命名
$targetFile = rtrim($targetPath,'/').'/'.iconv("UTF-8","GB2312",$_FILES["Filedata"]["name"]);

// Validate the file type
$fileTypes = array('jpg','jpeg','pjpeg','gif','png'); // File extensions
$fileParts = pathinfo($_FILES['Filedata']['name']);

if (in_array($fileParts['extension'],$fileTypes)) {
move_uploaded_file($tempFile,$targetFile);
echo '1';
} else {
echo 'Invalid file type.';
}
}
```