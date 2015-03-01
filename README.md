# ical-generator

[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](LICENSE)
[![CI Status](https://sebbo.helium.uberspace.de/teamcity-badges/ICalGenerator_Develop/status)](https://ci.sebbo.net/viewType.html?buildTypeId=ICalGenerator_Develop&guest=1)
[![Test Coverage](https://sebbo.helium.uberspace.de/teamcity-badges/ICalGenerator_Develop/coverage-istanbul)](https://ci.sebbo.net/viewType.html?buildTypeId=ICalGenerator_Develop&guest=1)

ical-generator is a small piece of code which generates ical calendar files. I use this to generate subscriptionable
calendar feeds.


## Installation

	npm install ical-generator


## Upgrade from 0.1.x

ical-generator 0.2.0 introduces a completely new API, but because you guys used 0.1.x a lot, the old API still works. So
you should be able to upgrade from ical-generator 0.1.x to 0.2.0 without any code changes. In case you need the old API
docs, you can find the deprecated documentation [here](https://github.com/sebbo2002/ical-generator/blob/0.1.10/README.md).

In case you have any issues with the new API, feel free to [create an issue](https://github.com/sebbo2002/ical-generator/issues/new).


## Quick Start

```javascript
var ical = require('ical-generator'),
	http = require('http'),
	cal = ical({domain: 'github.com', name: 'my first iCal'});

// overwrite domain
cal.domain('sebbo.net');

cal.createEvent({
	start: new Date(),
	end: new Date(new Date().getTime() + 3600000),
	summary: 'Example Event',
	description: 'It works ;)',
	location: 'my room',
	url: 'http://sebbo.net/'
});

http.createServer(function(req, res) {
	cal.serve(res);
}).listen(3000, '127.0.0.1', function() {
    console.log('Server running at http://127.0.0.1:3000/');
});
```


## Just another example

```javascript
var ical = require('ical-generator'),

    // Create new Calendar and set optional fields
    cal = ical({
        domain: 'sebbo.net',
        prodId: {company: 'superman-industries.com', product: 'ical-generator'},
        name: 'My Testfeed',
        timezone: 'Europe/Berlin'
    });

// You can also set values like this…
cal.domain('sebbo.net');

// … or get values
cal.domain(); // --> "sebbo.net"

// create a new event
var event = cal.createEvent({
    start: new Date(),
    end: new Date(new Date().getTime() + 3600000),
    timestamp: new Date(),
    summary: 'My Event',
    organizer: 'Sebastian Pekarek <mail@example.com>'
});

// like above, you can also set/change values like this…
event.summary('My Super Mega Awesome Event');

// get the iCal string
cal.toString(); // --> "BEGIN:VCALENDAR…"


// You can also create events directly with ical()
cal = ical({
    domain: 'sebbo.net',
    prodId: '//superman-industries.com//ical-generator//EN',
    events: [
        {
            start: new Date(),
            end: new Date(new Date().getTime() + 3600000),
            timestamp: new Date(),
            summary: 'My Event',
            organizer: 'Sebastian Pekarek <mail@example.com>'
        }
    ]
}).toString();
```



## API

### ical-generator

#### ical([Object options])

Creates a new [Calendar](#calendar) ([`ICalCalendar`](#calendar)).

```javascript
var ical = require('ical-generator'),
    cal = ical();
```

You can pass options to setup your calendar or use setters to do this.

```javascript
var ical = require('ical-generator'),
    cal = ical({domain: 'sebbo.net'});

// is the same as

cal = ical().domain('sebbo.net');

// is the same as

cal = ical();
cal.domain('sebbo.net');
```


### Calendar

#### domain([String domain])

Use this method to set your server's hostname. It will be used to generate the feed's UID. Default hostname is your
server's one (`require('os').hostname()`).


#### prodId([String|Object prodId])

Use this method to overwrite the default Product Identifier (`//sebbo.net//ical-generator//EN`). `prodId` can be ether
a valid Product Identifier or an object:

```javascript
cal.prodId({
	company: 'My Company',
	product: 'My Product',
	language: 'EN' // optional, defaults to EN
});

// OR

cal.prodId('//My Company//My Product//EN');
```


#### name([String name])

Use this method to set your feed's name. Is used to fill `X-WR-CALNAME` in your iCal file.


#### timezone([String timezone])

Use this method to set your feed's timezone. Is used to fill `X-WR-TIMEZONE` in your iCal.

```javascript
var cal = ical().timezone('Europe/Berlin');
```


#### createEvent([Object options])

Creates a new [Event](#event) ([`ICalEvent`](#event)) and returns it. Use options to prefill the event's attributes.
Calling this method without options will create an empty event.

```javascript
var ical = require('ical-generator'),
    cal = ical(),
    event = cal.createEvent({summary: 'My Event'});

// overwrite event summary
event.summary('Your Event');
```


#### events([Object events])

Add Events to calendar or return all attached events.

```javascript
var cal = ical();
cal.events([
    {
        start: new Date(),
        end: new Date(new Date().getTime() + 3600000),
        summary: 'Example Event',
        description: 'It works ;)',
        url: 'http://sebbo.net/'
    }
]);

cal.events(); // --> [ICalEvent]
```


#### save(String file[, Function cb])

Save Calendar to disk asynchronously using [fs.writeFile](http://nodejs.org/api/fs.html#fs_fs_writefile_filename_data_options_callback).


#### saveSync(String file)

Save Calendar to disk synchronously using [fs.writeFileSync](http://nodejs.org/api/fs.html#fs_fs_writefilesync_filename_data_options).


#### serve(http.ServerResponse response)

Send Calendar to the User when using HTTP. See Quick Start above.


#### toString()

Return Calendar as a String.


#### length()

Returns the ammount of events in the calendar.


#### clear()

Empty the Calender.



### Event

#### uid([String|Number uid]) or id([String|Number id])

Use this method to set the event's ID. If not set, an UID will be generated randomly.


#### start([Date start])

Appointment date of beginning as Date object. This is required for all events!


#### end([Date end])

Appointment date of end as Date object. This is also required for all events!


#### timestamp([Date stamp]) or stamp([Date stamp])

Appointment date of creation as Date object. Default to `new Date()`.


#### allDay([Boolean allDay])

When allDay == true -> appointment is for the whole day


#### floating([Boolean floating])
Appointment is a "floating" time. From [section 3.3.12 of RFC 554](https://tools.ietf.org/html/rfc5545#section-3.3.12):

> Time values of this type are said to be "floating" and are not
> bound to any time zone in particular.  They are used to represent
> the same hour, minute, and second value regardless of which time
> zone is currently being observed.  For example, an event can be
> defined that indicates that an individual will be busy from 11:00
> AM to 1:00 PM every day, no matter which time zone the person is
> in.  In these cases, a local time can be specified.


#### repeating([Object repeating])
Appointment is a repeating event

```javascript
cal.repeating({
    freq: 'MONTHLY', // required
    count: 5,
    interval: 2,
    until: new Date('Jan 01 2014 00:00:00 UTC')
});
```


#### summary([String summary])

Appointment summary, default to empty string.


#### description([String description])

Appointment description


#### location([String location])

Appointment location


#### organizer([String|Object organizer])

Appointment organizer

```javascript
cal.organizer({
    name: 'Organizer\'s Name',
    email: 'organizer@example.com'
});

// OR

cal.organizer('Organizer\'s Name <organizer@example.com>');
```


#### createAttendee([Object options])

Creates a new [Attendee](#attendee) ([`ICalAttendee`](#attendee)) and returns it. Use options to prefill the attendee's attributes.
Calling this method without options will create an empty attendee.

```javascript
var ical = require('ical-generator'),
    cal = ical(),
    event = cal.createEvent(),
    attendee = event.createAttendee({email: 'hui@example.com', 'name': 'Hui'});

// overwrite attendee's email address
attendee.email('hui@example.net');

// add another attendee
event.createAttendee('Buh <buh@example.net>');
```


#### attendees([Object attendees])

Add Attendees to the event or return all attached attendees.

```javascript
var event = ical().createEvent();
cal.attendees([
    {email: 'a@example.com', name: 'Person A'},
    {email: 'b@example.com', name: 'Person B'}
]);

cal.attendees(); // --> [ICalAttendee, ICalAttendee]
```


#### url([String url])

Appointment URL


#### method([String method])

Appointment method. May be any of the following: publish, request, reply, add, cancel, refresh, counter, declinecounter.


#### status([String status])

Appointment status. May be any of the following: confirmed, tenative, cancelled.



### Attendee

#### name([String name])

Use this method to set the attendee's name.


#### email([String email])

The attendee's email address. An email address is required for every attendee!


#### role([String role])

Set the attendee's role, defaults to `REQ-PARTICIPANT`. May be one of the following: req-participant, non-participant


#### status([String status])

Set the attendee's status. May be one of the following: accepted, tentative, declined


#### delegatesTo(ICalAttendee|Object attendee)

Creates a new Attendee if passed object is not already an attendee. Will set the delegatedTo and delegatedFrom attributes.

```javascript
var cal = ical(),
    event = cal.createEvent(),
    attendee = cal.createAttendee();

attendee.delegatesTo({email: 'foo@bar.com', name: 'Foo'});
```


#### delegatesFrom(ICalAttendee|Object attendee)

Creates a new Attendee if passed object is not already an attendee. Will set the delegatedTo and delegatedFrom attributes.

```javascript
var cal = ical(),
    event = cal.createEvent(),
    attendee = cal.createAttendee();

attendee.delegatesFrom({email: 'foo@bar.com', name: 'Foo'});
```



## Tests

```
npm test
```


## Copyright and license

Copyright (c) Sebastian Pekarek under the [MIT license](LICENSE).
