---
layout: post
title: "Leveraging Rails Partials To Render Content via AJAX"
date: 2014-05-29 19:24:34 -0400
comments: true
categories: 
---

Adding dynamic content to a page via AJAX can easily become a code intensive
experence. You may have seen it go down a little like this:

``` coffeescript
# app/assets/javascripts/weather.js.coffee

  eventHandler: (event) ->
    event.preventDefault()
   
    # Request JSON from rails
    $.ajax
      url: 'weather/fetch'
      type: 'GET'
      dataType: 'JSON'
      success: (data) ->
        window.currentWeather = data

        # Construct HTML string for new content
        htmlString += "<div id="temp"> #{Math.round(data.temp_f) + "F"} </div>"
        htmlString += "<div id="weather-icon">"
        htmlString += "<div class='" + data.icon + "-icon'></div>"
        htmlString += "</div>"

        # Insert new content into DOM 
        $("#weather-container").html(htmlString)
```

You can see that even a simple bit of content can quickly start to 
resemble the flying spagettii monster when attempting to construct it's HTML in this way.

You: Hey Rails?! I include your dumb ass for a reason, aren't you suppose to
take care of this shit for me???

Rails: I thought you'd never ask.

``` ruby
# config/routes.rb

get 'weather/fetch', to: 'api#fetch' 

```

``` ruby
# app/controllers/weather_controller.rb

def fetch
  @user_location ||= Location.new(request.remote_ip)
  data = WeatherUnderground::API.current_conditions("#{@user_location.latitude},#{@user_location.longitude}")
  render partial: "partials/weather", locals: {weather_data: data}
end

```

``` haml
// app/views/partials/_weather.haml

#temp
  = "#{data["temp_f"].round}F"
.icon{class: data["icon"]}
```

``` coffeescript
# app/assets/javascripts/weather.js.coffee

  eventHandler: (event) ->
    event.preventDefault()
   
    # Request HTML from rails partial we made
    $.ajax
      url: 'weather/fetch'
      type: 'GET'
      dataType: 'HTML'
      success: (data) ->
        # Insert new content into DOM 
        $("#weather-container").html(data.responseText)
```

I think this approach sets you up for creating much cleaner and
more maintainable code that will last for the long haul. Just add some slick
animations for a nice ease in effect.

Let me know in the comments your approach!
