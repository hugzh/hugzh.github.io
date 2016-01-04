title: php验证码制作
categories:
  - php
date: 2014-09-17 00:00:00
tags:
---

关于网页制作中的验证码部分是一个比较有趣的知识点，验证码的制作主要是基于PHP的图片处理拓展功能。 验证码制作的一般流程是：
1、创建验证码显示的底图
2、列出所有可能用到的字符内容以及字体颜色
3、随机截取部分（4或者5个）字符作为验证码输出
4、确定验证码显示的角度以及坐标
5、增加干扰元素（点和线条等）
6、php输出图片，并在脚本及时地destroy掉图片
<!--more-->
然后是一个值得注意的地方就是，一般来说验证码不应该出现容易混淆的字符，比如l和1,2和z,0和o等等，这是作为一个开发者从用户体验的角度来说需要注意的地方，另外就是干扰元素的颜色不应该使用鲜艳的颜色，一般来说颜色RGB值在50之后比较稳妥。
以下是我的php代码，命名为yanzhengma.php:

```php
$image = imagecreatetruecolor(100,30);//100x30的图片生成
$bgcolor = imagecolorallocate($image,255,255,255);//白色底色
imagefill($image,0,0,$bgcolor);`

//保存验证码内容便于服务器验证
$capchcode='';

for($i = 0;$i&lt;4;$i++){
$fontsize=12;
$fontcolor = imagecolorallocate($image,rand(0,120),rand(0,120),rand(0,120));

//$fontcotent = rand(0,9);//产生随机数字
$data = 'ABCDEHJKYSWTRabcdefghjkmnpqstuwsy23456789';
$fontcotent = substr($data,rand(0,strlen($data)),1);//随机截取一个字符
$capchcode .= $fontcotent;

$x = ($i*100/4)+rand(5,10);//定义内容的x坐标、y坐标，随机
$y = rand(5,12);
imagestring($image,$fontsize,$x,$y,$fontcotent,$fontcolor);//产生字符送到图片
}

$_SESSION['authcode'] = $capchcode;

//添加点干扰元素
for($i = 0;$i&lt;300;$i++){
$pointcolor = imagecolorallocate($image,rand(50,200),rand(50,200),rand(50,200));
imagesetpixel($image,rand(1,99),rand(1,29),$pointcolor);//点元素分布
}

//添加线条干扰
for($i = 0;$i&lt;3;$i++){
$linecolor = imagecolorallocate($image,rand(80,220),rand(80,220),rand(80,220));
imageline($image,rand(1,99),rand(1,29),rand(1,99),rand(1,29),$linecolor);//线条元素分布,两点确定一条直线
}
header('content-type:image/png');
imagepng($image);
imagedestroy($image);
```

总的来说，简单验证码的制作并不难，如果要做一些gif类型的验证码或者一些更加个性化的验证码就需要多花费一些时间去查找相关资料了，这里不做介绍。