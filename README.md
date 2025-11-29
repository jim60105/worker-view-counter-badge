# View Counter Badge

![View counter badge](https://view-counter.jim60105.workers.dev)

Count how many visitors any page gets. A simple solution to add a view counter badge to your readme files or web page.

A view counter implemented using Cloudflare Workers Edge storage solution.

> [!Note]
> I have rewritten this worker from KV storage to D1 storage.  
> The main reason is that the free plan **_limits KV to 1,000 write, delete, list operations per day_**, but **_allows D1 for 100,000 rows written per day_**.
>
> **The following are the differences from upstream**
>
> - Change from KV storage to D1 database.
> - Migrate project from JavaScript to TypeScript.
> - Migrate from Service Workers to ES Modules (D1 can only works with ES Modules).
> - Update CI workflow.

## Setup

The view counter badge is meant to be deployed individually for each profile/user. Click the button and then follow the steps below to deploy your own view counter. During the process, it will guide you to log in (or register) your GitHub and Cloudflare accounts.

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/jim60105/worker-view-counter-badge)

After completing the process, please return to GitHub. At this point, the Action will fail to deploy, so it is essential to proceed with the following configuration.

### Set your worker name

You can change the name in the first line of `wrangler.toml`, as it will serve as the worker name and is related to the subdomain.

Your Cloudflare Worker will be deployed at `{worker-name}.{cloudflare-id}.workers.dev`.

### Prepare your new D1 Database

Create a new D1 Database, it's important that you name it `ViewCounter`.

After obtaining the `database_id` from the execution result of the command, please insert it back into the `wrangler.toml` file and save the file. Following these steps:

#### Step 1: Create a new D1 Database

> [!IMPORTANT]  
> Execute the following command in your working directory.

Create a new D1 Database with the name `ViewCounter`.

```bash
wrangler d1 create ViewCounter
```

You will get a response with the `database_id` like this:

```bash
✅ Successfully created DB 'ViewCounter' in region APAC
Created your database using D1's new storage backend. The new storage backend is not yet recommended for production
workloads, but backs up your data via point-in-time restore.

[[d1_databases]]
binding = "DB" # i.e. available in your Worker on env.DB
database_name = "ViewCounter"
database_id = "72f5fd97-3f01-40e7-b31f-6c3117a9c624"
```

#### Step 2: Update `wrangler.toml`

Fill the `database_id` property in `wrangler.toml` with the id you got in the previous step.

> [!IMPORTANT]  
> `binding` and `database_name` must be `ViewCounter` for our code to work.

```toml
[[d1_databases]]
binding = "ViewCounter"
database_id = "72f5fd97-3f01-40e7-b31f-6c3117a9c624" # Your database id
database_name = "ViewCounter"
```

#### Step 3: Create the table

Create a table named `ViewCounter` with `init_database.sql`

```bash
wrangler d1 execute ViewCounter --file=./init_database.sql
```

## Deploy

Save the file and push a new commit into `master` and wait for the GitHub Action to deploy your worker.

## Add counter to README

```html
<img src="https://{worker-name}.{cloudflare-id}.workers.dev" alt="View counter badge" />
```

OR

```markdown
![View counter badge](https://{worker-name}.{cloudflare-id}.workers.dev)
```

## Customization

You can pass arguments to customize your view counter. The badge is created using `badgen` so all the options supported by `badgen` except for icon and iconWidth can be used.

|  Parameter   | Default |         Allowed values          |      Example       |
| :----------: | :-----: | :-----------------------------: | :----------------: |
|   `label`    |  Views  |           Any string            |  `label=Visitors`  |
| `labelColor` |   555   | RGB values or valid color names | `labelColor=black` |
|   `color`    |  blue   | RGB values or valid color names |   `color=green`    |
|   `style`    |  flat   |       `flat` or `classic`       |  `style=classic`   |
|   `scale`    |    1    |           Any integer           |     `scale=2`      |

Valid color names are:

```text
blue, cyan, green, yellow, orange, red, pink, purple, grey, black
```

Example of a counter with parameters:

```text
https://{worker-name}.{cloudflare-id}.workers.dev?style=classic&labelColor=black&color=green
```

## Multiple Deployments

You can deploy multiple view counters under the same GitHub account and Cloudflare account.

1. Create and checkout to a new git branch.
2. Change the `name` in `wrangler.toml` to deploy on a different subdomain.
3. Change the `CounterName` in `src/index.ts` to any other name you like. Please ensure that there are no conflicts with any existing names.
4. Commit and push to origin to trigger the GitHub workflow again.

## License

MIT License  
Copyright (c) 2022 Aveek Saha and 陳鈞

[![CodeFactor](https://www.codefactor.io/repository/github/jim60105/worker-view-counter-badge/badge)](https://www.codefactor.io/repository/github/jim60105/worker-view-counter-badge)
