## video标签动态加载src
~~~
<video :controls="true" autoplay playsinline webkit-playsinline>
   <source src="" type="application/x-mpegURL">
</video>
~~~
如上的html，会发现如果src被直接赋值的话,video是可以正常播放的，但如果是动态给src赋值会发现src值是有的，但没有视频可播放。可以推断出来的是，当 video 中存在 source 标签的时候，浏览器渲染之后会自动去获取地址，即便地址改变，浏览器也不会再去获取地址。

解决方式如下：
~~~
palyVideo() {
      var v = document.getElementsByTagName("video")[0];
      v.addEventListener('loadeddata', function(){
          v.play();
      });
      v.load();
    };
~~~
    
其中,loadeddata是在当前帧的数据是可用的时触发，相关事件如下：
loadstart
durationchange
loadedmetadata
loadeddata
progress
canplay
canplaythrough
