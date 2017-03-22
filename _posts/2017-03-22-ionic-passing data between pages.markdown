---
layout: 	post
title:		"Passing data between pages"
header-img:	"img/post-bg-deeplearning.jpg"
date:		2017-03-22
author: 	"Borg"
catalog:	true
tags:
    - Ionic
---

# Ionic 页面间传递数据
Ionic 页面间传递数据方式：

- NavController, NavParams
- Modal
- Provider
- Events

以下以页面A转换到页面B为例

## NavController

在页面A中通过NavConroller实例将B页面push进页面桟，push第二个参数为传递的数据。

```javascript
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';
import { PageB } from '../pageB/pageB'

@Component({
  selector: "page-A",
  templateUrl:"A.html"
})
export class PageA{
  constructor(public navCtrl: NavController){}
  openPageB() {
    navCtrl.push(PageB, {title:"hello ionic"});
  }
}
```

在页面B中通过NavParams获取数据如下：

```javascript
import { Component } from '@angular/core';
import { NavController, NavParams } from 'ionic-angular';

@Component({
  selector: "page-B",
  templateUrl:"B.html"
})
export class PageB{
  constructor(public navCtrl: NavController, public navParams: NavParams){}
  ionViewDidLoad() {
    console.log(this.navParams.get("title"));
  }
}

```

## Modal
附文档： [ModalController API](https://ionicframework.com/docs/v2/api/components/modal/ModalController/)

### A to B:
省略模块声明，具体看上方文档。页面A中通过ModalController实例的create函数第二个参数发送数据。

```js
let profileModal = this.modalCtrl.create(Profile, { userId: 8675309 });
profileModal.present();
```

B页面中同样可以使用NavParams获取数据

```js
constructor(params: NavParams) {
   console.log('UserId', params.get('userId'));
 }
```

### B to A:
当关闭B时传递数据回给页面A，首先在页面A中设置好B关闭后的回调函数。

```js
let profileModal = this.modalCtrl.create(Profile, { userId: 8675309 });
profileModal.onDidDismiss(data => {
  console.log(data);
});
profileModal.present();
```

页面B用ViewController关闭Modal，在dismiss函数第二个参数设置传递的数据。

```js
import { ViewController } from 'ionic-angular';

let data = { 'foo': 'bar' };
this.viewCtrl.dismiss(data);
```

## Provider
provider 功能同 Angularjs 1 中的service相同，在 App Module 设置好provider(不设置则每个子模块使用的service是不同实例，数据会不一致)，则子模块使用的是provider的同一实例，可以用来传递数据。可以在命令行使用  
ionic g provider providername  
生成模板，具体说明见 Angular 2.0 的[文档](https://angular.io/docs/ts/latest/tutorial/toh-pt4.html)。记得在App Module的provider里加上声明的service。


```
import { Injectable } from '@angular/core';

@Injectable()
export class HeroService {

  getData() {
      return {title:"hello ionic"}
  }
}
```

## Events
[Events Doc](https://ionicframework.com/docs/v2/api/util/Events/)  
以下是官方文档的代码，通过Events传递数据。

```js
import { Events } from 'ionic-angular';

constructor(public events: Events) {}

// 页面A
function createUser(user) {
  console.log('User created!')
  events.publish('user:created', user, Date.now());
}

// 页面B
events.subscribe('user:created', (user, time) => {
  // user and time are the same arguments passed in `events.publish(user, time)`
  console.log('Welcome', user, 'at', time);
});
```


# Resources
[Josh Morony: Passing Data Between Pages in Ionic 2](https://www.youtube.com/watch?v=T5iGAAypGBA&list=PLvLBrJpVwC7ocO1r-xu218C15iE9gTWBA&index=11)  
[Josh Morony: A Simple Guide to Navigation in Ionic 2](https://www.joshmorony.com/a-simple-guide-to-navigation-in-ionic-2/)
