## 前端JS常用工具函数

 在工作中需要的或是在其他地方遇见的比较不错的工具函数，长期更新
 
 
*  base64编码

```
let base64EncodeChars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
let base64DecodeChars = new Array(-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 62, -1, -1, -1, 63, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, -1, -1, -1, -1, -1, -1, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, -1, -1, -1, -1, -1, -1, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, -1, -1, -1, -1, -1);
let base64Encode = function(str) {
    let out, i, len;
    let c1, c2, c3;
    len = str.length;
    i = 0;
    out = "";
    while (i < len) {
        c1 = str.charCodeAt(i++) & 0xff;
        if (i == len) {
            out += base64EncodeChars.charAt(c1 >> 2);
            out += base64EncodeChars.charAt((c1 & 0x3) << 4);
            out += "==";
            break;
        }
        c2 = str.charCodeAt(i++);
        if (i == len) {
            out += base64EncodeChars.charAt(c1 >> 2);
            out += base64EncodeChars.charAt(((c1 & 0x3) << 4) | ((c2 & 0xF0) >> 4));
            out += base64EncodeChars.charAt((c2 & 0xF) << 2);
            out += "=";
            break;
        }
        c3 = str.charCodeAt(i++);
        out += base64EncodeChars.charAt(c1 >> 2);
        out += base64EncodeChars.charAt(((c1 & 0x3) << 4) | ((c2 & 0xF0) >> 4));
        out += base64EncodeChars.charAt(((c2 & 0xF) << 2) | ((c3 & 0xC0) >> 6));
        out += base64EncodeChars.charAt(c3 & 0x3F);
    }
    return out;
}
```
* base64 解码

~~~
let base64EncodeChars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
let base64DecodeChars = new Array(-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 62, -1, -1, -1, 63, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, -1, -1, -1, -1, -1, -1, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, -1, -1, -1, -1, -1, -1, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, -1, -1, -1, -1, -1);
let base64Decode = function(str) {
   let c1, c2, c3, c4;
   let i, len, out;
   len = str.length;
   i = 0;
   out = "";
   while (i < len) {
       /* c1 */
       do {
           c1 = base64DecodeChars[str.charCodeAt(i++) & 0xff];
       }
       while (i < len && c1 == -1);
       if (c1 == -1) 
           break;
       /* c2 */
       do {
           c2 = base64DecodeChars[str.charCodeAt(i++) & 0xff];
       }
       while (i < len && c2 == -1);
       if (c2 == -1) 
           break;
       out += String.fromCharCode((c1 << 2) | ((c2 & 0x30) >> 4));
       /* c3 */
       do {
           c3 = str.charCodeAt(i++) & 0xff;
           if (c3 == 61) 
               return out;
           c3 = base64DecodeChars[c3];
       }
       while (i < len && c3 == -1);
       if (c3 == -1) 
           break;
       out += String.fromCharCode(((c2 & 0XF) << 4) | ((c3 & 0x3C) >> 2));
       /* c4 */
       do {
           c4 = str.charCodeAt(i++) & 0xff;
           if (c4 == 61) 
               return out;
           c4 = base64DecodeChars[c4];
       }
       while (i < len && c4 == -1);
       if (c4 == -1) 
           break;
       out += String.fromCharCode(((c3 & 0x03) << 6) | c4);
   }
   return out;

~~~

* 获取字符串的视觉长度：中文2，英文1

~~~
let getLength = function(str) {
		if ( typeof(str) != 'string' ) return 0;
	    let realLength = 0, len = str.length, charCode = -1;
	    for (let i = 0; i < len; i++) {
	        charCode = str.charCodeAt(i);
	        if (charCode >= 0 && charCode <= 128) realLength += 1;
	        else realLength += 2;
	    }
	    return realLength;
	};
~~~


* 计算年龄

~~~
/**
*
*  @param from 出生日期
*  @param to 设定的日期
*/
let getAge = function(from, to) {
		let str = new Date(from).Format("yyyy-MM-dd");
		let today = new Date(to).Format("yyyy-MM-dd");
		let birthDay = str.split("-");
		let sub_y = parseInt(today.getFullYear()) - parseInt(birthDay[0]);
		let sub_m = parseInt(today.getMonth())    - ( parseInt(birthDay[1]) - 1 );
		let sub_d = parseInt(today.getDate())     - parseInt(birthDay[2]);
		if ( sub_y < 0 ) {	return '未出生';
		}else if ( sub_y == 0 ) {
			if ( sub_m < 0 ) {	return '未出生';
			}else if ( sub_m == 0 ) {
				if ( sub_d < 0 ) {	return '未出生';
				}else if( sub_d == 0 ) {  return '刚出生';
				}else{	return '0个月'	}
			}else{
				if ( sub_d < 0 ) {	return( (sub_m - 1) + '个月' );
				}else{	return(sub_m + '个月' ) 	}
			}
		}else{
			if ( sub_m < 0 ) {
				if (sub_y - 1 == 0) {
					if ( sub_d < 0 ) {	return( (11 + sub_m) + '个月' );
					}else{	return( (12 + sub_m) + '个月' )	}
				}else{
					let tBirth = str.split("-");
					tBirth[0] = parseInt(tBirth[0]);
					tBirth[0] += (sub_y - 1);
					let month_day = calc_age(tBirth[0]+'-'+tBirth[1]+'-'+tBirth[2], today);
					if ( month_day == '未出生' || month_day == '刚出生' || month_day == '0个月') {
						month_day = '整';
					}	
					return( (sub_y - 1) + '岁' );
				}
			}else if ( sub_m == 0 ) {
				if ( sub_d < 0 ) {
					if (sub_y - 1 == 0) {	return( 11 + '个月' );
					}else{
						let tBirth = str.split("-");
						tBirth[0] = parseInt(tBirth[0]);
						tBirth[0] += (sub_y - 1);
						let month_day = calc_age(tBirth[0]+'-'+tBirth[1]+'-'+tBirth[2], today);
						if ( month_day == '未出生' || month_day == '刚出生' || month_day == '0个月') {
							month_day = '整';
						}					
						return( (sub_y - 1) + '岁' );
					}
				}else{
					let tBirth = str.split("-");
					tBirth[0] = parseInt(tBirth[0]);
					tBirth[0] += sub_y;
					let month_day = calc_age(tBirth[0]+'-'+tBirth[1]+'-'+tBirth[2], today);
					if ( month_day == '未出生' || month_day == '刚出生' || month_day == '0个月') {
						month_day = '整';
					}
					return( sub_y + '岁' + month_day )	;
				}
			}else{
				let tBirth = str.split("-");
				tBirth[0] = parseInt(tBirth[0]);
				tBirth[0] += sub_y;
				let month_day = calc_age(tBirth[0]+'-'+tBirth[1]+'-'+tBirth[2], today);
				if ( month_day == '未出生' || month_day == '刚出生' || month_day == '0个月') {
					month_day = '整';
				}
				return( sub_y + '岁' + month_day );
			}
		}
	}
~~~

* 计算距离，参数分别为第一点的纬度，经度；第二点的纬度，经度

~~~
/**
*
*  @param lat1 维度
*  @param lng1 经度
*  @param lat2 维度
*  @param lng2 经度
*/
let getDistance = function(lat1,lng1,lat2,lng2) {
		function _rad(d) {
		   return d * Math.PI / 180.0; //经纬度转换成三角函数中度分表形式。
		}
	    let radLat1 = _rad(lat1);
	    let radLat2 = _rad(lat2);
	    let a = radLat1 - radLat2;
	    let  b = _rad(lng1) - _rad(lng2);
	    let s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a/2),2) +
	    Math.cos(radLat1)*Math.cos(radLat2)*Math.pow(Math.sin(b/2),2)));
	    s = s *6378.137 ;// EARTH_RADIUS;
	    s = Math.round(s * 10000) / 10000; //输出为公里
	    s = s*1000;
	    return s.toFixed(0);
	}
~~~

* 禁止右键、选择、复制

~~~

['contextmenu', 'selectstart', 'copy'].forEach(function(ev) {
	document.addEventListener(ev, function(event) {
		return event.returnValue = false;
	})
});
~~~

* 禁止某些键盘事件

~~~
document.addEventListener('keydown', function(event) {
    return !(
    	112 == event.keyCode ||		//禁止F1
    	123 == event.keyCode ||		//禁止F12
    	event.ctrlKey && 82 == event.keyCode ||		//禁止ctrl+R
    	event.ctrlKey && 18 == event.keyCode ||		//禁止ctrl+N
    	event.shiftKey && 121 == event.keyCode ||  		//禁止shift+F10
    	event.altKey && 115 == event.keyCode ||		//禁止alt+F4
    	"A" == event.srcElement.tagName && event.shiftKey		//禁止shift+点击a标签
    ) || （event.returnValue = false）
});
~~~

* performance.timing:利用performance.timing进行性能分析

~~~
window.onload = function() {
	setTimeout(function() {
		let t = performance.timing;
		console.log('DNS查询耗时 ：' + (t.domainLookupEnd - t.domainLookupStart).toFixed(0))
        console.log('TCP链接耗时 ：' + (t.connectEnd - t.connectStart).toFixed(0))
        console.log('request请求耗时 ：' + (t.responseEnd - t.responseStart).toFixed(0))
        console.log('解析dom树耗时 ：' + (t.domComplete - t.domInteractive).toFixed(0))
        console.log('白屏时间 ：' + (t.responseStart - t.navigationStart).toFixed(0))
        console.log('domready时间 ：' + (t.domContentLoadedEventEnd - t.navigationStart).toFixed(0))
        console.log('onload时间 ：' + (t.loadEventEnd - t.navigationStart).toFixed(0))
        if (t = performance.memory) {
			console.log('js内存使用占比：' +  (t.usedJSHeapSize / t.totalJSHeapSize * 100).toFixed(2) + '%')
		}
	})
}
~~~

* 判断一个数组是否包含一个指定的值，如果是返回true，否则返回false，可指定开始查询的位置

~~~
/**
*
*  @param value 数组
*  @param start 开始的下标
*/
Array.prototype.includes = Array.prototype.includes || function includes(value, start) {
	let ctx = this;
	let length = ctx.length;
	start = parseInt(start)
	if(isNaN(start)) {
		start = 0
	} else if (start < 0) {
		start = -start > length ? 0 : (length + start);
	}
	let index = ctx.indexOf(value);
	return index >= start;
}
~~~

* 取数组中非NaN数据中的最小值

~~~
function min(arr){
    arr = arr.filter(item => !_isNaN(item))
    return arr.length ? Math.min.apply(null, arr) : undefined
}
~~~

* 求取数组中非NaN数据中的最大值
~~~
function max(arr){
    arr = arr.filter(item => !_isNaN(item))
    return arr.length ? Math.max.apply(null, arr) : undefined
}
~~~

* 全屏
~~~
function toFullScreen() {
	let elem = document.body;
    elem.webkitRequestFullScreen
    ? elem.webkitRequestFullScreen()
    : elem.mozRequestFullScreen
    ? elem.mozRequestFullScreen()
    : elem.msRequestFullscreen
    ? elem.msRequestFullscreen()
    : elem.requestFullScreen
    ? elem.requestFullScreen()
    : alert("浏览器不支持全屏");
}
~~~

* 退出全屏
~~~
function exitFullscreen() {
	let elem = parent.document;
	elem.webkitCancelFullScreen
	? elem.webkitCancelFullScreen()
	: elem.mozCancelFullScreen
	? elem.mozCancelFullScreen()
    : elem.cancelFullScreen
    ? elem.cancelFullScreen()
    : elem.msExitFullscreen
    ? elem.msExitFullscreen()
    : elem.exitFullscreen
    ? elem.exitFullscreen()
    : alert("切换失败,可尝试Esc退出");
}
~~~
* 获取Url参数，返回一个对象
~~~
function GetUrlParam(){
    let url = document.location.toString();
    let arrObj = url.split("?");
    let params = Object.create(null)
    if (arrObj.length > 1){
        arrObj = arrObj[1].split("&");
        arrObj.forEach(item=>{
            item = item.split("=");
            params[item[0]] = item[1]
        })
    }
    return params;
}
~~~
*getPropByPath：根据字符串路径获取对象属性：‘obj[0].count’
~~~
function getPropByPath(obj, path, strict) {
    let tempObj = obj;
    path = path.replace(/\[(\w+)\]/g, '.$1'); //将[0]转化为.0
    path = path.replace(/^\./, ''); //去除开头的.

    let keyArr = path.split('.'); //根据.切割
    let i = 0;
    for (let len = keyArr.length; i < len - 1; ++i) {
        if (!tempObj && !strict) break;
        let key = keyArr[i];
        if (key in tempObj) {
            tempObj = tempObj[key];
        } else {
            if (strict) {//开启严格模式，没找到对应key值，抛出错误
                throw new Error('please transfer a valid prop path to form item!');
            }
            break;
        }
    }
    return {
        o: tempObj, //原始数据
        k: keyArr[i], //key值
        v: tempObj ? tempObj[keyArr[i]] : null // key值对应的值
    };
};
~~~
* isStatic: 检测数据是不是除了symbol外的原始数据

~~~
function isStatic(value) {
    return (
        typeof value === 'string' ||
        typeof value === 'number' ||
        typeof value === 'boolean' ||
        typeof value === 'undefined' ||
        value === null
    )
}
~~~
* 数组去重，返回一个新数组
~~~
function unique(arr){
    if(!isArrayLink(arr)){ //不是类数组对象
        return arr
    }
    let result = []
    let objarr = []
    let obj = Object.create(null)

    arr.forEach(item => {
        if(isStatic(item)){//是除了symbol外的原始数据
            let key = item + '_' + getRawType(item);
            if(!obj[key]){
                obj[key] = true
                result.push(item)
            }
        }else{//引用类型及symbol
            if(!objarr.includes(item)){
                objarr.push(item)
                result.push(item)
            }
        }
    })

    return result
}
~~~
* 使用Object.assign可以钱克隆一个对象：
~~~
let clone = Object.assign({}, target);
~~~