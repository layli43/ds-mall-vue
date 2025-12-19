

This project is the frontend part of a front-end and back-end separated architecture.

### Project Introduction

mall-app-web is a mobile e-commerce system built with uni-app.
It mainly includes features such as home portal, product recommendations, product search, product display, shopping cart, order process, member center, customer service, and help center.

### Technology Stack
| Technology   | Description                 | Official Website                                                                   |
| ------------ | --------------------------- | ---------------------------------------------------------------------------------- |
| Vue          | Core frontend framework     | [https://vuejs.org](https://vuejs.org)                                             |
| Vuex         | Global state management     | [https://vuex.vuejs.org](https://vuex.vuejs.org)                                   |
| uni-app      | Mobile frontend framework   | [https://uniapp.dcloud.io](https://uniapp.dcloud.io)                               |
| mix-mall     | E-commerce project template | [https://ext.dcloud.net.cn/plugin?id=200](https://ext.dcloud.net.cn/plugin?id=200) |
| luch-request | HTTP request framework      | [https://github.com/lei-mu/luch-request](https://github.com/lei-mu/luch-request)   |

```
Project Structure
src -- Source code directory
├── api -- luch-request network request definitions
├── components -- Common reusable components
├── js_sdk -- Third-party SDK source code
├── static -- Static resources such as images
├── store -- Vuex state management
├── utils -- Utility classes
└── pages -- Frontend pages
    ├── address -- Address management pages
    ├── brand -- Product brand pages
    ├── cart -- Shopping cart pages
    ├── category -- Product category pages
    ├── coupon -- Coupon pages
    ├── index -- Home page
    ├── money -- Payment pages
    ├── notice -- Notification pages
    ├── order -- Order pages
    ├── product -- Product pages
    ├── public -- Login pages
    ├── set -- Settings pages
    ├── user -- Member pages
    └── userinfo -- Member information pages
```