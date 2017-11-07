---
title: Teach you to recognize MVC, MVP and MVVM
date: 2017-03-26 16:12:29
tags: [模式、Android]
category: "总结"
---
### Teach you to recognize MVC, MVP AND MVVM
I believe we are no stranger to MVC, MVP and MVVM, as the three most familiar Android framework. Their applications can be very extensive, but for some novice, it can be difficult to distinguish between them three.
### Article focus:
1. Learn and distinguish between MVC, MVP and MVVM.
2. Know how these three models in the use of Android.
3. Out of the DataBinding errors.
4. Understand the development model of MVP data binding.

### MVC 
MVC means Model View Controller, is the most common framework in the software architecture, simply through the control of controller to operate the modle layer of data, and return to view layer display.

![Alt text](http://zjutkz.net/images/%E9%80%89%E6%8B%A9%E6%81%90%E6%83%A7%E7%97%87%E7%9A%84%E7%A6%8F%E9%9F%B3%EF%BC%81%E6%95%99%E4%BD%A0%E8%AE%A4%E6%B8%85MVC-MVP%E5%92%8CMVVM/mvc.png "Optional title")

Use the traditional MVC, which View, corresponding to a variety of layout files, but these layout files are not as powerful as the Web Side, can do very limited. Controller correspods to the Activity, but Activity has the function of operating UI. We in the actual project will hava a lot of UI operations, and also do a lot of View should be dong in the layer

MVC there is an important flaw, we look at the picture above, view layer and the model layer is mutual konw, which means that there is coupling between the two layers. The Coupling is very fatal for large program, beacuse it means that development, testing, maintenance need to spend a lot of energy.

### MVP
MVP as MVC evolution, to solve a lot of MVC shortcommings. For Android, MVP model layer relative to the MVC is the same, but activity and fragment is no longer the controller layer, but the pure view layer.And all fordwarding of user events is handled by the presenter layer.
![Alt text](http://zjutkz.net/images/%E9%80%89%E6%8B%A9%E6%81%90%E6%83%A7%E7%97%87%E7%9A%84%E7%A6%8F%E9%9F%B3%EF%BC%81%E6%95%99%E4%BD%A0%E8%AE%A4%E6%B8%85MVC-MVP%E5%92%8CMVVM/mvp.png "Optional title")

It can be seen from the figure that the most obvious difference is that the view and model layer are no longer mutual konw and complete descoupling. Instead of the precenter layer acts as bridge, the events used to manipulate the view layer are passed to the presenter layer, the presenter layer to mainpulate the model layer, and the data is returned to the view layer. the view layer and the model layer are completely unconnected throughout the process.

### MVVM
MVVM最早是由微软提出的
![Alt text](http://zjutkz.net/images/%E9%80%89%E6%8B%A9%E6%81%90%E6%83%A7%E7%97%87%E7%9A%84%E7%A6%8F%E9%9F%B3%EF%BC%81%E6%95%99%E4%BD%A0%E8%AE%A4%E6%B8%85MVC-MVP%E5%92%8CMVVM/mvvm.png "Optional title")

From the figure to see that it is similiar to MVP, but the presenter layer 
be replaced by ViewModel layer. And the view layer and the viewmodel are bound to each other, the means that when you update the data of the viewmodel layer, the view layer changes accordingly ui. 

### Last: MVP + Databinding
We use the data binding framework to save similar findViewById and data binding time, and the use of presenter to separate the business logic and view layer.