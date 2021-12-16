# ![](https://raw.githubusercontent.com/bamzi/gojobrunner/master/views/runclock.jpg) gojobrunner

gojobrunner is framework for performing work asynchronously, outside of the request flow. It comes with cron to schedule and queue job functions for processing at specified time. 

It includes a live monitoring of current schedule and state of active jobs that can be outputed as JSON or Html template. 

## Install

`go get github.com/rachitnovostack/gojobrunner`

### Setup

```go
package main

import "github.com/rachitnovostack/gojobrunner"

func main() {
    gojobrunner.Start() // optional: gojobrunner.Start(pool int, concurrent int) (10, 1)
    gojobrunner.Schedule("@every 5s", ReminderEmails{})
}

// Job Specific Functions
type ReminderEmails struct {
    // filtered
}

// ReminderEmails.Run() will get triggered automatically.
func (e ReminderEmails) Run() {
    // Queries the DB
    // Sends some email
    fmt.Printf("Every 5 sec send reminder emails \n")
}
```

### Live Monitoring
![](https://raw.githubusercontent.com/bamzi/gojobrunner/master/views/gojobrunner-html.png)
```go

// Example of GIN micro framework
func main() {
    routes := gin.Default()

    // Resource to return the JSON data
    routes.GET("/gojobrunner/json", JobJson)

    // Load template file location relative to the current working directory
    routes.LoadHTMLGlob("../github.com/rachitnovostack/gojobrunner/views/Status.html")

    // Returns html page at given endpoint based on the loaded
    // template from above
    routes.GET("/gojobrunner/html", JobHtml)

    routes.Run(":8080")
}

func JobJson(c *gin.Context) {
    // returns a map[string]interface{} that can be marshalled as JSON
    c.JSON(200, gojobrunner.StatusJson())
}

func JobHtml(c *gin.Context) {
    // Returns the template data pre-parsed
    c.HTML(200, "", gojobrunner.StatusPage())

}

```
## WHY?
To reduce our http response latency by 200+%

gojobrunner was created to help us process functions unrelated to response without any delays to the http response. GoRoutines would timeout because response has already been processed, closing the instance window all together. 

Instead of creating a separate independent app, we wanted to save time and manageability of our current app by coupling-in the job processing. We did not want to have micro services. It's premature optimization.

If you have a web app or api service backend and want a job processing framework built into your app then gojobrunner is for you. Once you hit mass growth and need to scale, you can simply decouple you go-jobrunners into a dedicated app.

## Use cases
Here are some examples of what we use gojobrunner for:

* Send emails to new users after signup
* Sending push notification or emails based on specifics
* ReMarketing Engine - send invites, reminder emails, etc ...
* Clean DB, data or AMZ S3
* Sending Server stats to monitoring apps
* Send data stats at daily or weekly intervals

### Supported Featured
*All jobs are processed outside of the request flow*

* Now: process a job immediately
* In: processing a job one time, after a given time
* Every: process a recurring job after every given time gap
* Schedule: process a job (recurring or otherwise) at a given time


## Compatibility

gojobrunner is designed to be framework agnostic. So it will work with pure Go apps as well as any framework written in Go Language. 

*Verified Supported Frameworks*

* Gin
* Echo
* Martini
* Beego
* Revel (gojobrunner is modified version of revel's Jobs module)
* ...

**Examples & recipes are coming soon**

## Basics

```go
    gojobrunner.Schedule("* */5 * * * *", DoSomething{}) // every 5min do something
    gojobrunner.Schedule("@every 1h30m10s", ReminderEmails{})
    gojobrunner.Schedule("@midnight", DataStats{}) // every midnight do this..
    gojobrunner.Every(16*time.Minute, CleanS3{}) // evey 16 min clean...
    gojobrunner.In(10*time.Second, WelcomeEmail{}) // one time job. starts after 10sec
    gojobrunner.Now(NowDo{}) // do the job as soon as it's triggered
```
[**More Detailed CRON Specs**](https://github.com/robfig/cron/blob/v2/doc.go)

## Contribute

**Use issues for everything**

- Report problems
- Discuss before sending pull request
- Suggest new features
- Improve/fix documentation

## Credits
- [revel jobs module](https://github.com/revel/modules/tree/master/jobs) - Origin of gojobrunner
- [robfig cron v3](https://github.com/robfig/cron/tree/v3) - github.com/robfig/cron/v3
- [contributors](https://github.com/rachitnovostack/gojobrunner/graphs/contributors)

### Author 
**Bam Azizi**
Github: [@bamzi](https://github.com/bamzi)
Twitter: [@bamazizi](https://twitter/bamazizi)

#### License
MIT