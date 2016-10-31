# Getting Started

## Dependencies

- Postgres
- PhantomJS
- Redis

## Environment variables (set them yourself!)

1. Copy `.env.example` to `.env`.
2. Copy `.env.test.example` to `.env.test`

## Installation and startup (macOS)

- Ensure your AWS environment variables are set
- [Install Homebrew] (http://brew.sh/)
- ``brew install gpg``
- ``brew install postgres``
- ``brew install redis``
- ``brew install phantomjs``
- ``postgres -D /usr/local/var/postgres``
- ``createuser -s postgres``
- Install [rbenv] (https://github.com/rbenv/rbenv) OR [rvm] (https://rvm.io/)
- Using rbenv or rvm, install ruby 2.3.1 (or version specified in Gemfile)
- ``bin/setup``
- Install [nvm] (https://github.com/creationix/nvm)
- ``nvm use``
- ``npm install``

To run the server locally:

- ``bundle exec rails server``
- ``npm run client:watch`` in another window
- ``redis-server --port 6374``

or

- `foreman start -f Procfile.dev`

In order to run foreman, you'll need to install the gem with `gem install foreman`

## Development Process
Currently, our process follows a flavor of [Extreme Programming](https://en.wikipedia.org/wiki/Extreme_programming). We keep track of all stories (features / bugs / chores) in [Pivotal Tracker](https://www.pivotaltracker.com/n/projects/1588965). These stories are typically written by the Product Manager and prioritized by the Product Owner. At the beginning of each week, we have an Iteration Planning Meeting with the whole team to discuss the stories for that week and estimate their complexity.

Once you've started a story, create a branch with the following naming convention, as well as the story ID:

- `features/{story-name}-1235`
- `bugs/{story-name}-1235`
- `chores/{story-name}-1235`

When you've completed a story, open a pull request with a screenshot or information about what has been changed. Once the PR has been approved, you should [rebase](https://git-scm.com/docs/git-rebase) your branch against `development` and merge via the Github interface (or via a no-ff merge). [See an example here](https://github.com/indigo-ag/core-web/pull/169)

Once merged, [CircleCI](https://circleci.com/gh/indigo-ag/core-web) will automatically run all the tests and deploy the application to our [acceptance](https://gft-acceptance.indigoag.com/) environment.

## Deployment
We are hosting our application on AWS Elastic Beanstalk, using a single EC2 instance with auto-scaling, Cloudwatch monitoring, and a postgres RDS instance for the database.

Merging into the origin `master` branch will deploy to [production](https://gft.indigoag.com/) once the CircleCI build passes, while any changes the origin `development` branch are deployed to our [acceptance](https://gft-acceptance.indigoag.com/) environment via CircleCI.

If you'd like to deploy a branch locally (assuming it is not `development` or `master`), ensure that your AWS user is permitted to deploy (and set as the default profile), then follow these steps:

- ``brew install awsebcli`` or use [pip](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html) to install the cli.
- ``eb init`` from the project root, which will give you options for which environment you'd like to deploy to.
- ``./bin/build`` to properly build static assets for a production environment.
- ``eb deploy`` to zip the application and deploy to ElasticBeanstalk!

In order to deploy to production, please follow these steps after ensuring you have no local changes:

```shell
$ git checkout master
$ git pull origin master
$ git merge origin/development --no-ff
$ git tag v{VERSION_NUMBER}
$ git push origin master --tags
```

On the `#releases` Slack channel, alert the team that a release is in progress, providing the version number, as well as any new features or bug fixes. We tag each release based on [Semantic Versioning](http://semver.org/).

## CSS Comb
Before committing any css changes, run the following to reformat your css. Run the command with `-lv` to just get lint errors

```
cd client
csscomb -c .csscomb  app
```

## Local SMTP (Mailcatcher)
The development environment will point SMTP traffic to localhost:1025.
We recommend spinning up Mailcatcher to serve the SMTP request, which
also allows you to inspect sent mail. This is not included in the
Gemfile, so we recommend just doing `gem install mailcatcher`.

## The Mock API
HTTP requests from within the app to `localhost:3000/mock_ag_int` are deferred via WebMock to the `MockAgInt` Sinatra service. This is used for emulating the behavior of the Ag Integrated API.

Records added to the API are persisted to Redis. Records can be added directly to the Redis store by using `MockAgInt::Fixtures`, which expects records to be added in the format matching the API schema.

The Mock API service does not run as a Rails route, so it is *not* accessible from your browser.

### How to stop using the Mock API

1. Open `.env`
1. Set `AG_INT_BASE_API_URL` to the address of the API root
1. If basic auth is used, you can also set the `BASIC_AUTH_AG_INTEGRATED_USERNAME` and `BASIC_AUTH_AG_INTEGRATED_PASSWORD` env vars

As long as the value doesn't contain `mock_ag_int` in the path name, WebMock will not be activated to intercept requests.

## Keeping Track of AgIntegrated API changes.
AgIntegrated has a swagger.yaml file which outlines what their API does/has available. We have copied this file into fixtures/swagger.yaml. During each test run we compare their living document to our snapshot. If they add endpoints, keys, query parameters  we should take care to manually create a new checked-in swagger.yaml file if we make use of those. Running the test with new additions should result in a changeset being displayed during the run. If they remove functionality our local test should fail.

To quickly replace the checked-in swagger.yaml with the remote version,
run `RAILS_ENV rake swagger:update`

## Running Tests

```
rake db:test:prepare
rake
```

This will run client specs, rspec unit tests, and rspec functional tests.

To just run the client specs, use `npm test` from the root directory.

If you'd like mocha to watch for changes, use `npm run test:watch`.

Tests are automatically run against the Mock API using a separate Redis store. RSpec will spin up and spin down the Redis instance.

### Screenshot specs

This project uses the [rspec-page-regression](https://github.com/rprt/rspec-page-regression/tree/v1.0) gem (at v1.0 branch). It allows for simple assertion writing against screenshot diffs generated by capybara's selected driver. New screenshot match assertions will return an error when run the first time -- the gem provides a command to copy the generated screenshot into version control. The workflow looks like:

1. Write new assertion
2. Run spec
3. Receive red test
4. Copy/paste provided command from output into your terminal
5. Screenshot is placed in version control
6. Run spec
7. Spec is green

When other developers run the spec on their environments, the screenshot generated will be compared to the screenshot placed in version control. If they match, the spec passes. If they don't match, the spec fails. Failures should be examined carefully to determine if the cause is regression, or legitimate feature change; the latter  means the generated screenshot should replace the screenshot in version control.

Variance may also occur due to environmental rendering differences. To help mitigate this, set the `RSPEC_PAGE_REGRESSION_THRESHOLD` value (e.g. `RSPEC_PAGE_REGRESSION_THRESHOLD=0.01` to allow for a 1% pixel variance). The more disparate environments that participate in this project, the more likely that all workstations will require this value.

Happy assertions normally focus on specific CSS selectors, rather than the whole page. For example, visualization assertions normally target _just_ the `<svg>` for the visualization.

Finally, this project vendors `rspec-page-regression` since its deploy cycle has haulted, and we're in need of edge features. We've also
modified its code to allow for a `:sleep` parameter. Providing this will force a sleep between viewport resizes and screenshots by the specified number of seconds. This is necessary for D3-enabled visualiation resizes.

### External services:

#### AWS S3

Visualization data "packets" (zip files) are uploaded to S3 by AgIntegrated. S3 is configured to trigger an SNS notification which pings an endpoint hosted on this app. This app will retrieve the specified file directly from S3.

In our test suite, no calls are made to S3, since the app does not upload files directly.

#### AWS SNS

Requests from SNS topic notifications _are mocked out_ in our test suite.

In order to test the SNS notifications locally, you'll need follow the following steps in AWS:

1. Create an S3 bucket.
2. Create an SNS topic and set its permission policy to allow your new S3 bucket access.
3. Modify the S3 bucket to trigger an `ObjectCreated` event and send it to the newly-created SNS topic.
4. Expose your localhost to a public url using a tool like [ngrok](https://ngrok.com/)
5. Create an subscription on the topic to point to the endpoint you want notifications sent to.
6. Grab the new topic ARN from AWS and set the env var `SNS_TOPIC_ARN` on your machine.

Once you've followed these steps, you can upload a "zip" file to your new bucket. The app will respond by unzipping the file, matching the csv file names to model names and loading the data into the database.

#### Mapbox

Invoked with the React-Leaflet library. In our test suite, we make real connections to the MapBox API, though we mock out tilesets. Just a good thing to know.

## Redux

We use the Redux pattern for developing complex front-end React
components.

Notes:

- The store is hydrated with API data via global `fetch` -- it is
  polyfilled with `whatwg-fetch`. Global `fetch` does not share the page
auth state by default, therefore `{ credentials: 'same-origin' }` is
provided with the `fetch` call.
