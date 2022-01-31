---
layout: post
title:  "PostNord delivery dates in Home Assistant"
date:   2022-01-31
---

Since Maud Olofsson bought the danish postal service with swedish tax money and merged it with the swedish equivalent the service has ever declined. The "new" company PostNord recently started to deliver mail on an every other day basis which makes it almost impossible to predict when to find bills and other fun important shipments in the letter-box.

Found that PostNord "offers" an API which can be used to get the next (and the one after that) delivey date so I integrated this with Home Assistant in order to trigger automations on dates where the mailman should be around. This is configured in 4 parts:

### The REST sensor
Fetches the data every hour from the API. Replace the postcode with your postcode of choice (if you aren't interested in when [the king](http://zverige.com/kingkong/) expects a delivery).
```
 - platform: rest
   name: PostMord
   resource: https://portal.postnord.com/api/sendoutarrival/closest
   scan_interval: 3600
   params:
     postalCode: 10770
   json_attributes:
      - postalCode
      - city
      - delivery
      - upcoming
```


### Mangle the REST result to ISO 8610
This creates two new sensors in the format of ISO 8601. `Delivery` is the closed date for delivery and `Upcoming` is the one after the that (nomenclature from their API).
{% raw %}
```
template:
  - sensor:
    - name: "PostMord Delivery"
      state: "{{ state_attr('sensor.postmord', 'delivery') |replace('januari', '01') |replace('februari', '02') |replace('mars', '03') | replace('april', '04') |replace('maj', '05') |replace('juni', '06') |replace('juli', '07') |replace('augusti', '08') |replace('september', '09') |replace('oktober', '10') |replace('november', '11') |replace('december', '12') |regex_replace(find='^(\\d{1,2})\\s(\\d{2}),\\s(\\d{4})', replace='\\g<3>-\\g<2>-\\g<1>', ignorecase=False)}}"
    - name: "PostMord Upcoming"
      state: "{{ state_attr('sensor.postmord', 'upcoming') |replace('januari', '01') |replace('februari', '02') |replace('mars', '03') | replace('april', '04') |replace('maj', '05') |replace('juni', '06') |replace('juli', '07') |replace('augusti', '08') |replace('september', '09') |replace('oktober', '10') |replace('november', '11') |replace('december', '12') |regex_replace(find='^(\\d{1,2})\\s(\\d{2}),\\s(\\d{4})', replace='\\g<3>-\\g<2>-\\g<1>', ignorecase=False)}}"
```
{% endraw %}

### Sensor based on date
This creates a sensor with the current date (in ISO 8610) for comparison below.
```
sensor:
 - platform: time_date
   display_options:
     - 'date'
```

### Binary sensor - mailday?
Compare the sensors from above in order to create a binary sensor which can be used as trigger or condition in automations.
{% raw %}
```
template:
  - binary_sensor:
    - name: PostMord Delivery today
      state: "{{ states('sensor.postmord_delivery') == states('sensor.date') }}"
```
{% endraw %}

Enjoy!
