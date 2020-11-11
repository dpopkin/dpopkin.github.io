---
layout: post
title:  "Laravel, type hinting and IDOR"
date:   2020-11-10 10:16:20 -0800
categories: Laravel
---

One of my favorite features of Laravel is something called [automatic injection](https://laravel.com/docs/7.x/container#automatic-injection). What it does is it allows us to automatically load a single ORM object by ID as an argument in a method. Consider the following:
{% highlight php %}
// routes/web.php
Route::get('price/{item}', ItemController@displayPrice);

// App/Http/ItemController.php
public function displayPrice(int $id)
{
  $item = Item::find(id)->price;
  return view('item.price', ['item' => $item]);
}
{% endhighlight %}

Can we simplify this? Yes, with automatic injection:
{% highlight php %}
public function displayPrice(Item $item)
{
  return view('item.price', ['item' => $item]);
}
{% endhighlight %}

Automatic injection greatly simplifies code, but should be used with caution. Consider an example involving a company that posts customer bills online:
{% highlight php %}
public function show(Bill $bill)
{
  return view('bill.show', ['bill' => $bill]);
}
{% endhighlight %}

Lets take a look at the route file in this example:
{% highlight php %}
Route::resource('bills', BillController::class)->only(['index', 'show'])->where(['bill' => '[0-9]+']);
{% endhighlight %}

In this example, we can see bill is set up as a typical Laravel [resource route](https://laravel.com/docs/8.x/controllers#restful-naming-resource-routes) with some validation on the bill parameter that checks to make sure it's a number, meaning the URL will look something like `/bills/{bill}`, where bill is an ID number.

I'm sure some of you already know where I'm going with this, but for everyone else, say the final url is `/bills/1`. What would happen if I changed the bill ID parameter to 2? Yep, I see bill number 2. This is known as [Insecure Direct Object Reference](https://portswigger.net/web-security/access-control/idor) or IDOR. Meaning that if I can change a number on a URL, I can access any bill I want.

Now how do we fix this? I'm sure there are plenty of hacks and workarounds, but in this case, the safest method is to abandon type hinting for [relationships](https://laravel.com/docs/8.x/eloquent-relationships#one-to-many) (why was that not done in the first place?). If you want to see this in action for yourself, I've created a [repository](https://github.com/dpopkin/DVLA) that has this issue and you can try it yourself. Just set it up and then go to the bills section to get started.