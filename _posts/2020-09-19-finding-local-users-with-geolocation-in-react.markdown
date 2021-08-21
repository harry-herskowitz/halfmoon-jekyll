---
layout: post
title: 'Connecting Local Users with Geolocation in React'
date: 2020-09-19
categories: [React, Node, JS]
image: 'geolocation.jpg'
author: 'Harry Herskowitz'
author_blurb: 'Harry is a web developer and videographer with a focus on using technology to empower local artists and communities'
author_link: 'https://github.com/roldyclark'
avatar: 'roldy.png'
---

I am currently working on a React app that helps content creators find and collaborate with other creators in their area. Thankfully, the Geolocation Web API makes getting a user's coordinates simple. I will walk through how I used this API to update a user's coordinates at login, and share a useful function I found for filtering users by distance.

First, I added the following code within the useEffect hook in App.js to prompt the user to allow location services:

```
if ('geolocation' in navigator) {
  console.log('geolocation supported')
} else {
  alert('no geolocation support')
}
```

Next, I called getCurrentPosition after the login action and dispatched the latitude and longitude to my geolocate action:

```
dispatch(login(email, password)).then(() => {
  navigator.geolocation.getCurrentPosition((position) => {
    dispatch(geolocate(
        position.coords.latitude,
        position.coords.longitude
    ))
  })
})
```

The geolocate action looks like this:

```
//Update Geolocation
export const geolocate = (latitude, longitude) => async (dispatch) => {
  try {
    const body = {
      latitude,
      longitude
    }
    await api.post('/users/geolocation', body)
  } catch (err) {
    const errors = err.response.data.errors
    if (errors) {
      errors.forEach((error) => dispatch(setAlert(error.msg, 'danger')))
    }
  }
}
```

Here is my post route (Express.js):

```
// @route    POST api/users/geolocation
// @desc     update user geolocation
// @access   Public

router.post('/geolocation', [auth], async (req, res) => {
  try {
    const user = await User.findById(req.user.id).select('-password')
    user.latitude = req.body.latitude
    user.longitude = req.body.longitude
    await user.save()
    res.json({ msg: 'Location Saved' })
  } catch (error) {
    console.error(error.message)
    res.status(500).send('Server error')
  }
})
```

I made sure to add latitude and longitude to my user model, and voila! I had the latest location of my users.
Now, I wanted to only show profiles within 30 km of the current user. I came across this helpful function which found the distance between two coordinate pairs:

```
function getDistance(lat1, lon1, lat2, lon2) {
  var R = 6371 // Radius of the earth in km
  var dLat = deg2rad(lat2 - lat1) // deg2rad below
  var dLon = deg2rad(lon2 - lon1)
  var a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(deg2rad(lat1)) *
      Math.cos(deg2rad(lat2)) *
      Math.sin(dLon / 2) *
      Math.sin(dLon / 2)
  var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a))
  var d = R * c // Distance in km
  return d
}

function deg2rad(deg) {
  return deg * (Math.PI / 180)
}
```

One filter later, and my task was complete:

```
{profiles.filter(
    (profile) =>
    getDistance(
        latitude,
        longitude,
        profile.user.latitude,
        profile.user.longitude
    ) < 30
).length > 0 ? (
    profiles.map((profile) => (
        <ProfileItem key={profile._id} profile={profile} />
 	))
 ) : (
 	<h4>No profiles found...</h4>
 )}
```
