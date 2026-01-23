# NOTES

## Jan 21

| Setting         | Current Value                    | New Value                    |
|-----------------|----------------------------------|------------------------------|
| Branch          | develop                          | main                         |
| Repo            | GoGeviti/geviti-emr-backend      | GoGeviti/geviti-web-app      |
| Root Directory  | /apps/geviti-emr-backend         | (empty)                      |
| Watch Patterns  | ["/apps/geviti-emr-backend/**"]  | (empty)                      |


* Update workspace get started doc
* Add more documentation on creating client id and client secret for infisical
* Fix postgres dump to use not use local libraries. Use docker
* Update EMR backend to use allow CORS env var
* Make a single source of truth for the ports
* ws status failing to detect if frontend are running
* make stripe CLI setup independent from local tools. Use docker
* add website postgres db to local


## Jan 15

https://linear.app/geviti/project/cicd-migration-45168eb52777/overview
I’ve created a Linear project and tickets to get us started with practicing Continuous Delivery (CD).

The definition of done for the project is just to set up the deployment pipeline. That’s essentially all you need to start practicing CD.

We won’t see the real benefits of CD until we have a deployment pipeline we trust, and that’s something we’ll build up gradually over time.

Workspace PR

- https://github.com/GoGeviti/geviti-web-app/pull/1601

CLOSED

- https://github.com/GoGeviti/geviti-emr-frontend/pull/354 CLOSED
- https://github.com/GoGeviti/geviti-emr-backend/pull/705 CLOSED
- https://github.com/GoGeviti/geviti-website-3/pull/25 CLOSED

## Jan 14

Workspace PR's

- https://github.com/GoGeviti/geviti-web-app/pull/1601
- https://github.com/GoGeviti/geviti-emr-frontend/pull/354
- https://github.com/GoGeviti/geviti-emr-backend/pull/705
- https://github.com/GoGeviti/geviti-website-3/pull/25

Here's some good resources on CI/CD

- https://minimumcd.org/
- https://www.youtube.com/watch?v=tQMrrNo16jo
