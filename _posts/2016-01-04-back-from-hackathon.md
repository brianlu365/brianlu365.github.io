---
layout: post
title: Back from AT&T Hackthon 2016
categories: [code]
tags: [hackathon, rails5, actioncable, IFTTT]
published: True

---

Hello 2016! Two events get me excited about every new year. AT&T Hackathon and CES.

I just got back from this year's AT&T Hackathon. It was a blast! Beside cool swags and prizes, I like the motivational atmosphere.

So I met my ex-coworker Jeremy and his two friends Grant and Anthony. Our idea was to promote recycling soda cans or anything with
a barcode to scan. Grant made a Andriod app to scan barcode using NantMobile ID API with the phone camera. It then send barcode and
user name to AT&T m2x cloud storage which triggers http post to my web server. User will get points for what he/she recycled. Users
can compete with points or redeem points for coupon and such.

Things I learned and did in the hackathon:

## rails 5
I did a rails 5 app. Yes! Using rails 5 beta 1. I want to try out actioncable to push real time notification once I received a new data point
on m2x. 

```ruby
# item controller
# example params[:title] '012512160,brian'
def collect
  string = params[:title]
  barcode, name = string.split(',')
  @item = RecyclableItem.where(barcode: barcode).first
  @user = User.where(name: name).first
  if @item and @user
    @user.recyclable_item << @item
    @user.save!
    ActionCable.server.broadcast 'bin_channel', name: @user.name, item: @item.name
    render json: {status: :ok, item: @item, user: @user}
  else
    render json: {status: :not_found}
  end
```

So once I saved the received recycable item, a broadcast will send to the bin channel.

```coffee
# assets/javascript/channel/bin.coffee
App.bin = App.cable.subscriptions.create "BinChannel",
  connected: ->
    # Called when the subscription is ready for use on the server

  disconnected: ->
    # Called when the subscription has been terminated by the server

  received: (data) ->
    alert data['name']
    # Called when there's incoming data on the websocket for this channel

  recycle: ->
    @perform 'recycle'
```

The browser will get a alert with user name. Cool!

Problem:

I got this working on my local development, but not in production. Since actionable is still new, there are not a lot info on how
to deploy it. From what I read so far, you will need to run two servers and listening on different port. one for web request and one for
websocket request.

## IFTTT

I knew what IFTTT is but never use it much. I used to get email about daily deals using IFTTT. It seems it has grown a lot with many
third party support. Such as you can implement logic for Amazon Echo. 

At the hackathon we couldn't define callback trigger for m2x on android, then the support staff there taught me how to use IFTTT to do that.

Here is an example https://ifttt.com/recipes/368284-at-t-m2x-trigger-example

## Conclusion

I probably slept less than 5 hrs on Saterday, my eyes hurts and I think I probably gained some weight for sitting so long and eating 
too much yummy food they provide. 

But Hackthon is awesome. I learned and did a lot in very short amound of time. I have won twice in the past. This is like the fourth time I attend. Even though 
we didn't win anything, I actually feel pretty good this time. It is a good test on my capability of learning and problem solving. 

I did many things for the first time:

* setup server for hosting on Digital Ocean 
* deploy with Capistrano https://www.digitalocean.com/community/tutorials/deploying-a-rails-app-on-ubuntu-14-04-with-capistrano-nginx-and-puma


