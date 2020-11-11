---
layout: post
title:  "Mass Assignment In Laravel"
date:   2020-11-11 14:12:00 -0800
categories: Laravel
---

[Mass assignment](https://en.wikipedia.org/wiki/Mass_assignment_vulnerability) is an issue in the Active Record pattern where a attribute that is not supposed to be assigned in a particular request (POST, PUT, PATCH, DELETE) is assigned and included in the processing of the request. Consider the following example where a store uses a Laravel application to keep track of items:
{% highlight php %}
// ItemRequest.php
public function rules()
{
    return [
        'aisle' => 'required|string',
        'price' => 'nullable|numeric',
        'description' => 'string|nullable|max:200'
    ];
}

// ItemController.php
public function update(ItemRequest $request, $id)
{
    $item = Item::find($id)->update($request->input());
    return back()->with('status', 'Item updated.');
}

// Item.php
class Item extends Model
{
    use HasFactory;

    protected $fillable = [
        'description', 'price', 'aisle'
    ];
}
{% endhighlight %}

In this scenario, we see a model that allows us to update the price, description and aisle of an item. Now assume there are two forms to update the item, let's check the view of the form used to update the item by a standard employee:
{% highlight html %}
<form method="POST" action="/items/{{$item->id}}">
    @csrf
    @method('PATCH')
    <div class="form-group">
        <label for="description">Description (200 characters max): </label>
        <input id="description" name="description" type="text" max-length="200" value="{{$item->description}}" class="form-control @error('description') is-invalid @enderror">
        
        @error('description')
            <div class="alert alert-danger">{{ $message }}</div>
        @enderror
    </div>
    <div class="form-group">
        <label for="aisle" class="">Aisle (200 characters max): </label>
        <input id="aisle" type="text" max-length="200" name="aisle" value="{{$item->aisle}}" class="form-control @error('aisle') is-invalid @enderror">

         @error('aisle')
             <div class="alert alert-danger">{{ $message }}</div>
        @enderror
    </div>
    <button type="submit" class="btn btn-primary">Submit</button>
</form>
{% endhighlight %}

No price, what gives? Turns out a standard employee is supposed to be unable to update the price from the form itself. But they can, thanks to price being defined as a fillable attribute in the model.

Now how to exploit? There are many options out there for altering POST requests, from [Postman](https://www.postman.com/) to [Burp Suite](https://portswigger.net/burp), but for this simple example, we will use [cURL](https://curl.se/). CURL is installed on most UNIX systems by default, making it a good first choice. 
1. First we need to open the developer tools to see what, where and how the form data is sent.
2. Navigate to the `network` tab and submit the form as normal.
3. (Chrome) Right/Control click the request, `copy -> copy as cUrl`.
4. (Firefox) same as Chrome, expect FF has a `edit and resend` feature that can replace what we want cURL to do.
5. Paste the copied cURL data into a text editor of choice and find out how the form request and data is sent. I won't give away everything, but it'll be easy to figure out.
6. Paste the cURL command into a terminal.
7. It'll respond with a 302 redirect request, but refreshing the page will show the price has been updated.

What about a fix? Well, admins are supposed to edit price, so what do we do? We can use the `authorize` method in the ItemRequest file. Consider this as a possible solution (assuming one admin with an ID of 1):
{% highlight php %}
// ItemRequest.php
public function authorize()
{
    if(request()->input('price')) {
        return Auth::id() === 1;
    }
    return true;
}
{% endhighlight %}

A ready-to-go version of this scenario is also included in my [DVLA](https://github.com/dpopkin/DVLA) project if you want to try it for yourself or have any ideas on how to make it even better.