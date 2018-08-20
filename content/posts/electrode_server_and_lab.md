---
title: "Electrode Server and Lab - effin eh!"
date: 2018-08-16T14:32:52-08:00
draft: false
---
Walmart labs released Electrode awhile back for universal JavaScript. The project uses "best of" technologies and presumably helps power a bunch of their stuff. How much? I have no idea as I do not work for Walmart labs. I did apply once but that is a different post... Anyways, back to electrode-server, this has been my server boilerplate since it was released. Matched with electrode-confippet (node-config) you have a server up and running in one line of code. The server does nothing but it would be running. 

```
const config = require("electrode-confippet").config;
//Exports a Promise
module.exports = require("electrode-server")(config);
```
Electrode-confippet is not a hard requirement but it seems to work really well and lets you use .js or .json for your configuration needs. I like the flexibility so I will stick with it. The rest of the electrode stack integrates nicely with React and if you are into React I would recommend that you go that way. I personally am not on team React so I opted to just use the server. As an aside, you could also use a more traditional Hapi setup and be just fine... 

The real hurdle I needed to overcome was what that line above does and how do I integrate lab into the mix. As far as I can tell electrode went the route of Mocha and friends. For this test I wanted to stick with lab as I am running a traditional(ish) REST api. On to the meat and potatoes of this. The file above exports a Promise so in our test file we need to remember that. 

```
//test/index.js
'use strict'

const Code = require('code')
const Hapi = require('hapi')
const Lab = require('lab')
const ElectrodeServer = require("../");

// shortcuts
const lab = exports.lab = Lab.script()
const describe = lab.describe
const it = lab.it
const expect = Code.expect

describe('electrode loaded echoIt correctly', () => {
  it('will echo my text', (done) => {
    var electrodeServer = ElectrodeServer
    electrodeServer.then( server => {
    server.inject('/electrode loaded this', (res) => {
        expect(res.result).to.equal('electrode loaded this')
        done()
      })
    server.stop()
   })

  })
})
```

For this example I loaded a Plugin called echoIt that literally just replies back with the input given. Notice how I call 
```
electrodeServer.then(server => {})
```
that gives us the server that we can inject into with lab and use as needed. In order to run this we just need to launch it via `npm test`

```
//package.json
  "scripts": {
    "test": "lab -c"
  }
```

Assuming you have more than one test you want to run you could spin up the server beforeEach or as needed. gg
