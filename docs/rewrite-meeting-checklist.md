# Rewrite meeting checklist

## Links

[Old Repo](https://github.com/GoGeviti/geviti-web-app)
[New Repo](https://github.com/GoGeviti/geviti)

## Goal

- New codebase should solve issues with the current codebase
- Safely migrate from old legacy codebase to the new codebase

## Problems with old codebase that new codebase needs to address

- Third party integration bugs
- Supporting multiple frontends between web app and flutter
- Lack of continuous delivery pipeline
- Lack of continuous integration
- Competing standards

## How todo it

- No new features on old app. Bug fixes only
- Use same exact datasources as old app like Postgres instance, Medplum instance, Stipe instance and so on...
- Parallel deployment with beta users as soon as possible
- Users should be able swap back and forth between both apps
