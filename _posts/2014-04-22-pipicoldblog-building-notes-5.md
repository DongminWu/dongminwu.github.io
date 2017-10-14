---
layout: post
title: pipicoldblog搭建手记5
date: 2014-04-22
categories: Blog
---

弄了两个上午。终于把GeSHi的代码高亮功能加到blog上面了

现在只需要在代码段中加入`<!@@php@@!>`

类似的字符串就能自动高亮了。

一下是收工后的总结：

恩就是玩弄字符串的一次操作

大概思路是找到所有的`<pre><code>`和`</pre></code>`对的坐标,分别放在两个数组里面

然后把文章的主题进行按照得到的坐标进行分割成一个`sub_content`数组

分隔后的每1,3,5....单数个数组元素都是代码了(因为每篇文章都有标题所以)`sub_content[0]`总是有东西的

然后就是用GeSHi处理一下再扔进去.

总之现在的blog显示页面`t.php`已经有自己独立的css提供样式了,

对了关于CSS显示`<pre>`标签.还遇到了一个不能自动换行的小问题,百度了一下找到了一篇文章

[使pre的内容自动换行](http://www.16sucai.com/2010/10/941.html)

大概就是加上个样式

 ```CSS
 <!@@css@@!>
 pre {
white-space: pre-wrap;
word-wrap: break-word;
}
 ```


- 有空记得我再把原来的代码全部加上`<!@@code@@!>`的tag吧~



下面就用高亮代码的代码显示高亮代码的代码(好绕口的一句话!!!~~~~)

```PHP
<!@@php@@!>
//highlight_func.php

//written by pipicold

include_once "./libs/geshi/geshi.php";


function highlight_article($blogcontent){

	$geshi=new GeSHi();

	$body_start=stripos($blogcontent, "<body");
	$body_end=strripos($blogcontent, "</body>");
	$content=substr($blogcontent, $body_start,$body_end+7-$body_start);



	$code_starts=array();
	$code_ends=array();
	$sub_contents=array();


	$start_point=0;
	$end_point=0;

	while (true) {

		$tmp = stripos($content, "<pre><code>",$start_point);

		if ($tmp == null){
			break;
		}else{

			array_push($code_starts, $tmp);
			$start_point=$tmp+1;
		}
	}

	while (true) {

		$tmp = stripos($content, "</code></pre>",$end_point);

		if ($tmp == null){
			break;
		}else{

			array_push($code_ends, $tmp);
			$end_point=$tmp+1;
		}
	}

	if (count($code_starts)==0){
		array_push($sub_contents, $content);
	}else{
		$sub_contents[0]=substr($content, 0,$code_starts[0]-1);
		for ($i=0; $i < count($code_starts) ; $i++) { //代码区一定在$sub_content中的2i+1的那个位置0<i<count($code_starts)
			$sub_contents[$i*2+1] = substr($content, $code_starts[$i],($code_ends[$i]+13-$code_starts[$i]));

			//---------parse_code code---start
			$language=null;
			$sub_contents[2*$i+1]=htmlspecialchars_decode($sub_contents[2*$i+1],ENT_QUOTES);//把html代码转成字符
			preg_match("/<!@@(.+?)@@!>/", $sub_contents[$i*2+1], $language);

			if ($language==null) {

			}else{
				$sub_contents[2*$i+1]=substr($sub_contents[2*$i+1], 11,-13);
				$geshi->set_language($language[1]);

				$sub_contents[2*$i+1]=str_ireplace("<!@@".$language[1]."@@!>", "", $sub_contents[2*$i+1]);

				$geshi->set_source($sub_contents[2*$i+1]);
				$sub_contents[2*$i+1]=$geshi->parse_code();
			}

			//---------parse_code code----end

			if ($code_starts[$i+1]==null){
			}else{
				$sub_contents[$i*2+2] =	substr($content, $code_ends[$i]+14,($code_starts[$i+1]-1-$code_ends[$i]-14));
			}

		}
		array_push($sub_contents, substr($content, $code_ends[count($code_ends)-1]+14));

	}

return implode("", $sub_contents);

}


```
