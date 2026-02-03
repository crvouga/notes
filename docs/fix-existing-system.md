# Fix existing system

## Links

[Repo](https://github.com/GoGeviti/geviti-web-app)

## Problems we're having

- Bugs
- Third party integration bugs
- Third party integration coupling
- Lack of flexibility
- Supporting multiple frontends between web app and flutter
- Competing standards
- Lack of documentation

### Secondary problems

- Messy database
- Messy code

## Solutions

- Move to a monorepo
- Rewrite parts of the app not the entire app.
- Delete old unused code
- Identify and rewrite the buggy parts of the app. Mainly the third party integrations
- Fix the causes not the symptoms
- Have a good CI pipeline
- Remove unused code
- Add documentation
- Clean up messy database
- Clean up messy code
