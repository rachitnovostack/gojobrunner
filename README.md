# ![](https://raw.githubusercontent.com/bamzi/go-jobrunner/master/views/runclock.jpg) go-jobrunner

go-jobrunner is framework for performing work asynchronously, outside of the request flow. It comes with cron to schedule and queue job functions for processing at specified time. 

It includes a live monitoring of current schedule and state of active jobs that can be outputed as JSON or Html template. 

## Install

`go get github.com/rachitnovostack/go-jobrunner`

### Setup

```go
package main

import "github.com/rachitnovostack/go-jobrunner"

func main() {
    go-jobrunner.Start() // optional: go-jobrunner.Start(pool int, concurrent int) (10, 1)
    go-jobrunner.Schedule("@every 5s", ReminderEmails{})
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
![](https://raw.githubusercontent.com/bamzi/go-jobrunner/master/views/go-jobrunner-html.png)
```go

// Example of GIN micro framework
func main() {
    routes := gin.Default()

    // Resource to return the JSON data
    routes.GET("/go-jobrunner/json", JobJson)

    // Load template file location relative to the current working directory
    routes.LoadHTMLGlob("../github.com/rachitnovostack/go-jobrunner/views/Status.html")

    // Returns html page at given endpoint based on the loaded
    // template from above
    routes.GET("/go-jobrunner/html", JobHtml)

    routes.Run(":8080")
}

func JobJson(c *gin.Context) {
    // returns a map[string]interface{} that can be marshalled as JSON
    c.JSON(200, go-jobrunner.StatusJson())
}

func JobHtml(c *gin.Context) {
    // Returns the template data pre-parsed
    c.HTML(200, "", go-jobrunner.StatusPage())

}

```
## WHY?
To reduce our http response latency by 200+%

go-jobrunner was created to help us process functions unrelated to response without any delays to the http response. GoRoutines would timeout because response has already been processed, closing the instance window all together. 

Instead of creating a separate independent app, we wanted to save time and manageability of our current app by coupling-in the job processing. We did not want to have micro services. It's premature optimization.

If you have a web app or api service backend and want a job processing framework built into your app then go-jobrunner is for you. Once you hit mass growth and need to scale, you can simply decouple you go-jobrunners into a dedicated app.

## Use cases
Here are some examples of what we use go-jobrunner for:

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

go-jobrunner is designed to be framework agnostic. So it will work with pure Go apps as well as any framework written in Go Language. 

*Verified Supported Frameworks*

* Gin
* Echo
* Martini
* Beego
* Revel (go-jobrunner is modified version of revel's Jobs module)
* ...

**Examples & recipes are coming soon**

## Basics

```go
    go-jobrunner.Schedule("* */5 * * * *", DoSomething{}) // every 5min do something
    go-jobrunner.Schedule("@every 1h30m10s", ReminderEmails{})
    go-jobrunner.Schedule("@midnight", DataStats{}) // every midnight do this..
    go-jobrunner.Every(16*time.Minute, CleanS3{}) // evey 16 min clean...
    go-jobrunner.In(10*time.Second, WelcomeEmail{}) // one time job. starts after 10sec
    go-jobrunner.Now(NowDo{}) // do the job as soon as it's triggered
```
[**More Detailed CRON Specs**](https://github.com/robfig/cron/blob/v2/doc.go)

## Contribute

**Use issues for everything**

- Report problems
- Discuss before sending pull request
- Suggest new features
- Improve/fix documentation

## Credits
- [revel jobs module](https://github.com/revel/modules/tree/master/jobs) - Origin of go-jobrunner
- [robfig cron v3](https://github.com/robfig/cron/tree/v3) - github.com/robfig/cron/v3
- [contributors](https://github.com/rachitnovostack/go-jobrunner/graphs/contributors)

### Author 
**Bam Azizi**
Github: [@bamzi](https://github.com/bamzi)
Twitter: [@bamazizi](https://twitter/bamazizi)

#### License
MIT