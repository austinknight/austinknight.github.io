---
layout: post
title: Making a Weather App with AngularJS and $HTTP Requests
modified: 2014-06-04
tags: [angular, $http]
categories: [how-to]
comments: true
pinned: false
---

AngularJS has some awesome stuff out of the box that allows for making data requests to an API a breeze. With the $http module, we can make an Asynchronous Javascript and XML (aka AJAX) request in only a few lines of code. If you haven't used Angular yet, making requests with $http is similar to the jQuery $.ajax() method used for requests. We give some configurations and receive a promise in return; mostly the same thing in Angular.

For this example, I'll assume that you already know how to get your application bootstrapped with AngularJS and have a basic understanding of things like $scope, binding data, services/factories, and controllers. If not, you can read up here [https://docs.angularjs.org/tutorial/](https://docs.angularjs.org/tutorial/). I'll include getting the app set up since this is my first Angular post though. You can see the [demo](#demo) at the bottom of the post.

As an example, we're going to build miniature application that will allow us to look up the weather anywhere in the United States. We'll use a factory to fetch our data from an api, then we'll add a controller to set our data to the $scope so we can use it. We will then make practical use of our data to search for weather by zip code.

#### Setting up the app
First, let's bootstrap our app with Angular. Include the angular script within the head tag and add a new file in your js directory called app.js, this is where we will invoke our app. This also uses the angular served js file, which shouldn't be used in a production environment.

{% highlight html %}
  <!DOCTYPE html>
  <html>
    <head>
      <meta charset="utf-8" />
      <title>AngularJS Plunker</title>
      <script>document.write('<base href="' + document.location + '" />');</script>
      <link rel="stylesheet" href="style.css" />
      <script data-require="angular.js@1.2.x" src="https://code.angularjs.org/1.2.16/angular.js" data-semver="1.2.16"></script>
      <script src="app.js"></script>
    </head>
    <body>
    </body>
  </html>
{% endhighlight %}

Inside of the app.js file, start by adding a new variable called `app` and set it to the angular module method with the first argument being the name of the application, which I am calling 'weatherApp', and the second to an empty array. The empty array is where dependencies that your app may rely on get injected. More on this to come in our data factory.

{% highlight javascript %}
  angular.module('weatherApp', []);
{% endhighlight %}

Now that we have our app set, include the app.js file in your html, it can go after the angular.js script. Then we will attach our app to the document by adding the `ng-app` attribute to the html tag, setting it equal to the name of the app we just created - like this:

{% highlight html %}
  <html ng-app="weatherApp">
{% endhighlight %}

#### Getting the data
Now we have a functioning angular application. On to the good stuff, let's set up a factory to fetch the data we need to find the weather. Back in the app.js file, we can use our app variable to call our application to add a new factory using the `.factory()` method. Adding a new factory is similar to how we created our app, the first argument is the name we want to give the factory and the second argument is an array of our dependency modules given as strings. This will be a little different however because we will add a function as our last dependency. This is what gets executed whenever our factory is called up and injects the dependencies. The function also takes in the same sibling dependencies that the function resides in. You can read up more about the reason for this structure here, and has mostly to do with minification issues [https://docs.angularjs.org/tutorial/step_05#-prefix-naming-convention](https://docs.angularjs.org/tutorial/step_05#-prefix-naming-convention). Finally, the factory will return an object with keys that we can use in our controller to call the function we'll make.

{% highlight javascript %}
  app.factory('weatherService', ['$http', '$q', function ($http, $q){
    return {

    };
  }]);
{% endhighlight %}

With our factory created, we can add some functionality to it. We will only need one function to make our GET request to the api, and it will be called `getWeather` which is returned in our object for use wherever we decide to inject weatherService. In the code above, you can see $http and $q were passed in as dependencies. We can use Angular's built in $http service to make our get request and $q to create the promise we return. Below is the full function used to make a request to our api for some data. We will either get a success or an error, either one returning some sort of promise with data or an error.

{% highlight javascript %}
  app.factory('weatherService', ['$http', '$q', function ($http, $q){
    function getWeather (zip) {
      var deferred = $q.defer();
      $http.get('https://query.yahooapis.com/v1/public/yql?q=SELECT%20*%20FROM%20weather.forecast%20WHERE%20location%3D%22' + zip + '%22&format=json&diagnostics=true&callback=')
        .success(function(data){
          deferred.resolve(data.query.results.channel);
        })
        .error(function(err){
          console.log('Error retrieving markets');
          deferred.reject(err);
        });
      return deferred.promise;
    }

    return {
      getWeather: getWeather
    };
  }]);
{% endhighlight %}

Let's break this down a bit. The getWeather function takes in a zip code that get passed to the url we send in our request. The first line in the function sets our $q dependency equal to a more readable name that we will use inside our api call. Next, we make our call to the api via the get method on the $http service. Since this is a public api, we only need to specify the url to the data we want. No matter what the outcome, the $http request will always return a promise. If we get some data returned from the server, we will enter the success function which resolves our promise with the data requested; otherwise we will get some sort of error messaging and the promise will get rejected. That's it for this service, we can now use this to get some data through our controller that we will be able to set on the scope for interacting with by the user.

####Passing data along
Setting up a controller is similar to setting up a factory. We will instead used the `.controller()` method on our app variable. In the controller we'll need the $scope and the weatherService module we just made to inject as dependencies. Our first line in the controller will make a call to the weatherService's getWeather function that will be invoked when a private function in our controller is called. This does not have to be wrapped in a private function, but we will do so because we want to use this action in multiple places within the controller and we don't want to repeat ourselves. Since we expect to get a promise back after calling `getWeather()`, we will chain the `.then()` method to it and pass in the data that is returned. Once our call is done and the promise has resolved or rejected, we bind the data to the $scope giving it the name 'place'. The weatherService function should be called as soon as the controller is loaded onto the page, so we can call our private function on the next line using fetchWeather(). We will initially make a call to the api for my zip code, 84105, in Salt Lake City. You can put in whatever numbers you want for displaying a default when the page is loaded, otherwise there will be no data. If you don't want any data initially, just leave the argument blank.

{% highlight javascript %}
  app.controller('weatherCtrl', ['$scope', 'weatherService', function($scope, weatherService) {
    function fetchWeather(zip) {
      weatherService.getWeather(zip).then(function(data){
        $scope.place = data;
      });
    }

    fetchWeather('84105');
  }]);
{% endhighlight %}

After the above code is in place, we will add a new $scope variable called findWeather and set it equal to a function that takes the zip code as an argument. We can use findWeather() now to attach the functionality to the submit button of the form we will make to look up an area; more to come on this next. This new function on the $scope will essentially do everything the other function does, except it will first clear out any date set for $scope.place so that we don't have a bunch of data that doesn't relate to the zip code we are giving. We will use our private function again to call up our new weather location, but clear out the $scope date first.

{% highlight javascript %}
  app.controller('weatherCtrl', ['$scope', 'weatherService', function($scope, weatherService) {
    function fetchWeather(zip) {
      weatherService.getWeather(zip).then(function(data){
        $scope.place = data;
      });
    }

    fetchWeather('84105');

    $scope.findWeather = function(zip) {
      $scope.place = '';
      fetchWeather(zip);
    };
  }]);
{% endhighlight %}

#### Using data in the view
This should be all the javascript we need to get the data we want to display to the user. We can now bind information in the html using Angular's binding syntax. Back in the index.html file, we'll need to attach our controller to the section of the code we want to be able to bind data in. Attaching the controller is similar to attaching the app to the document, instead we will use the ng-controller attribute and set it equal to the name of our controller `weatherCtrl`. We can place this on the body tag and our controller will be able to set data on any child element. Generally, you want to bind the controller to more specific areas so it cannot be used everywhere, but since this is a demo we'll make it easy and bind to the body.

{% highlight html %}
  <body ng-controller="weatherCtrl">
  <!--Need some weather stuff-->
  </body>
{% endhighlight %}

To see the data that gets returned from our api call, we can use the Angular binding syntax and call up our $scope.place variable with `{{ place }}` and insert it within the body tag. You should see a bunch of json data. We can use anything in that set to display to the user, awesome! In our app we want a user to be able to enter a specific zip code they are interested in. We'll need to create a form with a number input that gets submitted to our api call address. To get the number from the input, we can use the ng-model attribute and set it equal to any variable name, I am calling it zip. We will now have a variable on the $scope called zip that will have the value of whatever we type into the field. On our submit button, we can attach the `findWeather()` function we made in our controller by using the ng-click attribute and setting it equal to our function with `zip` being our argument. Remember, zip is the information we get from our input. Whenever we click submit, our `findWeather()` function will now be called with whatever we have entered and the data we receive will be set on the $scope variable `place`. We can now use place with whatever keys that are available in our json data object. You can look at the additional markup I've added to get an idea of how you can work with stuff. I've added a title for the location, the current weather, and the forecast for the next few days.

{% highlight html %}
  <body ng-controller="weatherCtrl">
    <form>
      <input type="number" ng-model="zip">
      <input type="submit" value="Search" ng-click="findWeather(zip)">
    </form>
    <p ng-show="zip">Searching the forecasts for: {{zip}}</p>
    <div>
      <h1>Forecast For: {{ place.location.city }}</h1>
      <h2>Current: {{ place.item.condition.text }} | {{ place.item.condition.temp }}&deg; <img src="http://l.yimg.com/a/i/us/we/52/{{place.item.condition.code}}.gif"/></h2>
      <div ng-repeat="forecast in place.item.forecast">
        <p><strong>{{ forecast.date }}</strong> – {{ forecast.text }}</p>
        <p>H: {{ forecast.high }}&deg; | L: {{ forecast.low }}&deg;</p>
      </div>
    </div>
  </body>
{% endhighlight %}

That's all for this little app. You can play around with the data bindings and arrange the markup however you like to display different information. If you have any questions leave a comment and I'll try to help you out. Check out the demo below to see it in action!

#### Demo
<iframe id="demo" width="100%" height="500px" src="http://embed.plnkr.co/SsUCnE"></iframe>
