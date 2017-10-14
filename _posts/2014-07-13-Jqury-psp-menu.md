---
layout: post
title: 老子回来了!带着jQuery
date: 2014-07-13
categories: Javascript

---




说起来真的好久没更新blog了

上一次的文章日期还是5月16日.

于是快过了2个月过来报道一下!

唔这两个月对于技术方面真的是很少更新.

上篇blog更新完了之后就忙着写毕业论文和帮媳妇儿做她的毕业设计

大概是一个FLASH+Arduino的一个展示性的项目

唔它重要的一点就是serproxy吧, 有空的话我再把源文件上传到git吧

之后就是毕业季和大家玩耍然后和基友们白白和老婆分居两地><

一个人来到上海找房子安顿下来开始上班,然后最近陷入了一点对未来的迷茫之中...

回归正题,这两天下班的时候想了想不能那么无聊是吧~于是自己学了一下jQuery这个框架怎么使用,发现它确实比原生的Javascript要好用和方便许多~!!!!

大概就属于随意敲敲代码就能弄出漂亮网页应用

目前还是一个完成了一半的状态


先是展示



就是一个点击就能切换二级菜单的东西..

唔用的原理是仿造于**图片轮播**的

然后自己加上了一些按键控制和主菜单动画

### 源代码
{%highlight javascript%}

	<!@@javascript@@!>
    /**
     * Created by Wu on 14-7-6.
     */


    //    2 vector test success
    //    var aa=new Array();
    //    aa[0]=new Array();
    //    aa[0][0]=10;
    //    alert(aa[0][0]);
    //



    var Main_index=0;//当前主按钮序号
    var Child_index=0;//当前子按钮序号
    var animate_time=200;//每个动画的执行时间
    var main_panel_button_counts=0;
    var child_panel_counts= new Array();
    var child_now_index=new Array();
    var can_press_key=true;



    $(document).ready(function(){

    	//init
    	main_panel_button_counts=$("li.buttons").length;//count the sum of button in main panel
    	count_child_menu();
    	init_child_now_index();
    	main_panel_change(Main_index);
    	can_press_key=true;

        $(document).keydown(function(event){
    	if(can_press_key==true){
    	    log("keyCode:"+event.keyCode+" is clicked.");
            if (event.keyCode==37){
               //left key down
    	   if (Main_index-1 >=0 ){
    		main_panel_change(Main_index-1);
    	   }
            };
            if (event.keyCode==39){
                //right key down
    	    if (Main_index+1<main_panel_button_counts){
    	    	main_panel_change(Main_index+1);
    	    }
            }
            if (event.keyCode==38){
                //up key down
    		if(child_now_index[Main_index]>0){
    			child_panel_change(child_now_index[Main_index]-1);
    		}
            }
    	if (event.keyCode==40){
    		//down key down
    		if (child_now_index[Main_index]<child_panel_counts[Main_index]-1){
    			child_panel_change(child_now_index[Main_index]+1);
    		}
    	}
    	}


        });
    //funciton main_panel_change(destination)
    //this function is used to move main panel,
    //when key down or mouse click;
    //let $("li.buttons) move to destination
    //and run the animate.
    //


        function main_panel_change(destination){
    	can_press_key=false;
    	log("can_press_key:"+can_press_key);
    	//change the child panel


    	//=================bug 1 when click the keys quickly, display will be wrong===========
    	//$("li.mymenu").eq(0).fadeOut(animate_time);
    	//$("li.mymenu").eq(1).fadeOut(animate_time);
    	//$("li.mymenu").eq(2).fadeOut(animate_time);
    	//$("li.mymenu").eq(4).fadeOut(animate_time);
    	//$("li.mymenu").eq(5).fadeOut(animate_time);
    	$("li.mymenu").eq(Main_index).fadeOut(animate_time,function(){$("li.mymenu").eq(destination).fadeIn(animate_time);can_press_key=true;});

    	//change main panel background-color
    	for (var i=0;i<main_panel_button_counts;i++){
    		if (i==destination){
    			$("li.buttons").eq(i).css("background-color","#e0e0e0");
    		}else{
    			$("li.buttons").eq(i).css("background-color","transparent");
    		}
    	}

    	//move main panel

    	var now_left=$("div#main_menu_panel").css("left");
            var move_distance=100;

            $("div#main_menu_panel").animate({left:"-="+(destination-Main_index)*move_distance+"px"})

            //let Main_index=destination
           	Main_index=destination;
    	child_panel_change(child_now_index[destination]);
        }

        //function child_panel_change()
        //when key down up and down, call this
        //1. get Main_index number to decide change which one
        //2
        function child_panel_change(destination){
        	log("child_destination :"+destination);
    	//change main panel background-color
    	for (var i=0;i<child_panel_counts[Main_index];i++){
    		if (i==destination){
    			$("ul.popupmenu").eq(Main_index).children("li").eq(i).css("background-color","#e0e0e0");
    		}else{
    			$("ul.popupmenu").eq(Main_index).children("li").eq(i).css("background-color","transparent");
    		}
    	}
    	child_now_index[Main_index]=destination;
        }



        //function count_child_menu()
        //a for loop to init the child_panel_counts[]
        function count_child_menu(){
    	for (var i=0;i<main_panel_button_counts;i++){
    		child_panel_counts[i]=$("ul.popupmenu").eq(i).children("li").length;
    	}
        }
        //function init_child_now_index()
        //init child_now_index to all 0
        function init_child_now_index(){
    	for (var i=0;i<main_panel_button_counts;i++){
    		child_now_index[i]=0;
    	}
        }
        //function log(data)
        //to output @ <p id="logoutput"></p>
        function log(data){

    //	$("#logoutput").text($("#logoutput").text()+"\r\n"+data);
        }
    //----------------------------below this line is mouse click event-------------------

        //$("#logoutput").text("start test");
        $("li.buttons").click(function(){

            var now_index=$(this).index();
        	main_panel_change(now_index);
    		});

        //子菜单被点击事件
        $("ul.popupmenu li").click(function(){

          //  alert("index:"+);
          var now_child_index=$(this).index();
          	child_panel_change(now_child_index);

        });

    });

{%endhighlight%}


### 稍微分析一下吧

主要的两个函数是**`main_panel_change`**和**`child_panel_change`**这两个分别是主菜单和子菜单的控制函数,

貌似没什么好讲的都挺简单><
