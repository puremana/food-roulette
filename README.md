# Random Food Picker

A fun little web app written in [Vue](https://github.com/vuejs/vue) that picks a random food business around your location using the [Yelp Fusion API](https://www.yelp.com/fusion).

Blog post on the process of creating the application [https://blog.jeremyshaw.co.nz/2018/11/19/CORS-And-Hiding-Api-Keys/](https://blog.jeremyshaw.co.nz/2018/11/19/CORS-And-Hiding-Api-Keys/)

![Random Food Picture Screenshot](random-food-picker.jpg)

## Commands

Project setup
```
npm install
```

Compiles and hot-reloads for development
```
npm run serve
```

Compiles and minifies for production
```
npm run build
```

## Customization

Boolean `filter` field in App.vue data controls whether or not all of the random food picker filters are shown. Current set to false due to lack of support of many of the filters in many areas of the world, most notably in Europe.
