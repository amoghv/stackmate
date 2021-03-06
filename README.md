
# Stackmate - CloudFormation for CloudStack

A library designed to read existing CloudFormation templates 
and execute them on a CloudStack deployment. 
Uses the [ruote](http://ruote.rubyforge.org) workflow engine,
and embeds a modular [Sinatra](http://www.sinatrarb.com/) application for wait handles.

Unlike CloudFormation, it does not  run as a web application. 
Instead it runs everything on the client side.

[Stacktician](https://github.com/chiradeep/stacktician) embeds stackmate to run it as a web application.

Note that only Basic Zone (aka EC2-Classic) is supported for now.

Follow:
* \#cloudstack-dev on Freenode
* <http://cloudstack.apache.org/mailing-lists.html>
* [@chiradeep](http://twitter.com/chiradeep) on Twitter

## Dependencies

stackmate uses [Bundler](http://gembundler.com/) to setup and maintain its
environment. Before running stackmate for the first time you need to install
Bundler (gem install bundler) and then run:

```bash
$ bundle install

```

Bundler will download all the required gems and install them for you.

Have a look at the Gemfile if you want to see the various dependencies.

## Getting started quickly

### Using the source

* Get the source files using git

```bash
$ git clone http://github.com/chiradeep/stackmate.git
$ cd stackmate
```

* Make sure every dependency is resolved.

```bash
$ bundle install
```
* Find your API key and secret key and the url for CloudStack

For example

```bash
$ export APIKEY=upf7L-tvcHFCSYhKhw-45l9IfaKXNQSWf0nXyWye6eqOBpLT5TqN8XQGeuloV3LbSwD6zuucz22L233Nrqg2pg
$ export SECKEY=9iSsuImdUxU0oumHu0p11li4IoUtwcvrSHcU63ZHS_y-4Iz3w5xPROzyjZTUXkhI9E7dy0r3vejzgCmaQfI-yw
$ export URL="http://localhost:8080/client/api"
```

* The supplied templates are taken from the AWS samples. 

You need a couple of mappings from AWS ids to your CloudStack implementation:

```bash
$ cat local.yml 
---
service_offerings:
  m1.small: 1c8db272-f95a-406c-bce3-39192ce965fa
templates:
  ami-1b814f72: 3ea4563e-c7eb-11e2-b0ed-7f3baba63e45
zoneid: b3409835-02b0-4d21-bba4-1f659402117e
```

* Ensure you have a ssh keypair called 'Foo' (used in the template parameter below) for your account FIRST:

```bash
$ cloudmonkey
☁ Apache CloudStack 🐵 cloudmonkey 4.1.0-snapshot3. Type help or ? to list commands.

> create sshkeypair name='Foo'
```


* Create a LAMP stack:

```bash
bin/stackmate MYSTACK01   --template-file=templates/LAMP_Single_Instance.template -p "DBName=cloud;DBUserName=cloud;SSHLocation=75.75.75.0/24;DBUsername=cloud;DBPassword=cloud;DBRootPassword=cloud;KeyName=Foo"
```

* If everything is successful, stackmate will hang after deploying the security groups and vms. 
You should see an output like this:

```bash
Your pre-signed URL is: http://localhost:4567/waitcondition/20130425-0706-kerujere-punopapa/WaitHandle
Try: curl -X PUT --data 'foo' http://localhost:4567/waitcondition/20130425-0706-kerujere-punopapa/WaitHandle
```
Executing the curl should unblock the wait handle. The idea of course is that the instance boots up, and reads its userdata and calls the same URL.

If you don't want the wait condition server to run, just use '-n'. Stackmate will not hang with this flag.

If you want to validate your template, you can use the --dry-run option. This will parse and validate the template and create an execution schedule.

StackMate allows you to define your own participants that will be called based on the template namespace. For example, you can define a class Foo::Bar
and use Foo::Bar as a "Type" in the template. The requirement is :
(1) It should contain a class variable @@stackmate_participant set to true so as to register with StackMate
(2) Foo::Bar should have Ruote::Participant as its ancestor
(3) Foo::Bar should have a method named "consume_workitem" that defines actions to be taken when called with workitem.

Use --plugins <Directories with ruby files> to add plugins. This has undergone limited testing


## TODO
* Parallelize (with ruote concurrence) where possible
* rollback on error
* timeouts
* embed in a web app ( [Stacktician](https://github.com/chiradeep/stacktician) )
* add support for Advanced Zone templates (VPC), LB, etc

## Feedback & bug reports

Feedback and bug reports are welcome on the [mailing-list](dev@cloudstack.apache.org), or on the `#cloudstack-dev` IRC channel at Freenode.net.

## License

(The MIT License)

Copyright (c) 2013 Chiradeep Vittal

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## libraries used

- ruote, <http://ruote.rubyforge.org/>
- sinatra, <http://www.sinatrarb.com/>
- cloudstack_ruby_client, <https://github.com/chipchilders/cloudstack_ruby_client>

Many thanks to the authors 

