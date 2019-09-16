# Mess with Node in Docker

I want to set up a Node application that runs in Docker and that I can debug from VS Code.

Then I'd like to implement both development and production branches of this behavior. I think that will mean multi-stage Docker builds for compact deployment containers.

Then I'd like to try this with a Next.js app, which changes the nature of how you build and run.

Check my commit messages to see the tutorial articles I'm following.

## Plans

- [x] Node app in Docker
- [x] Debuggable in VS Code
- [x] Debug and Release container variants
- [x] Next.js
- [x] TypeScript
- [x] Allow launching Storybook at the test stage
- [x] Allow running CI tools at the test stage
- [x] Allow debugging individual tests

## How this setup uses Docker

- `npm run compose:debug` uses `docker-compose.yml` to prepare a test environment including a debuggable web server and a Storybook server.
- `npm run compose:ci` uses `docker-compose.ci.yml` to prepare a test environment for continuous integration jobs.
- `npm run compose:release` uses `docker-compose.release.yml` to prepare a release archive that runs a production web server.
- `Dockerfile` describes a multi-stage build.
    - The test stage includes a copy of the checked out workspace with all packages installed. The builds for the CI and debug services stop here.
    - The build stage continues by removing development packages and preparing a release build.
    - The archive stage starts fresh and copies in just what's needed at runtime for a small release container.

## How to use this setup

- Clone the repo.
- Install Docker and VS Code.
- Open `app.code-workspace`. VS Code will recommend a Docker plugin if you don't have it, so install that.
- To build and run, right-click `docker-compose.yml` for debug mode or `docker-compose.release.yml` for release mode and select Compose Up.
    - Alternatively, use `npm run compose:debug up --build` or `npm run compose:release up --build`.
    - Canonically, `docker-compose up --build` will start debug mode too.
    - In VS Code's activity bar, click the Docker icon to view running containers.
    - With either application running, visit localhost:3000 to view the site.
    - With the debug application running, visit localhost:6006 to view Storybook. It may take a moment for the Storybook server to start after the container launches.
    - With the debug application running, save a code file to hot-reload the browser.
    - With the debug application running, select Start Debugging from the Debug menu to attach the debugger. Then click in VS Code's gutter to set breakpoints. In this example, only the web server is debuggable, not Storybook.
    - To stop, right-click again and select Compose Down, or use the `down` command the same way you used `up`.
- To run tasks in Docker as in continuous integration:
    - Bring up the CI service with `npm run compose:ci up --build`, or skip this step if you don't need to rebuild.
    - Run tests with `npm run test:unit:ci`.
    - Remove the service with `npm run compose:ci down`.
- In addition to the Docker debugger attachment launcher, VS Code launchers also exist for debugging locally, including the web server, all unit tests, and a single unit test file.
