# Next.js with AWS Lambda

This is a tiny example to show how to deploy the Next.js app on AWS Lambda using [Apex Up](https://up.docs.apex.sh/#introduction).

- Post : https://moondaddi.dev/post/2019-01-11-Deploy-Nextjs-app-to-AWS-Lambda/

> You may encounter the cold start issue, then refer to the lambda warmer.
> https://github.com/mattdamon108/lambda-warmer

- Post : https://moondaddi.dev/post/2019-03-23-how-to-warm-lambda-function/

## Next.js app with custom server

The custom server for Next.js app is needed to run your app on AWS Lambda. In this example, `express` will be used.

```javascript
// server.js

const express = require("express");
const next = require("next");

const port = parseInt(process.env.PORT, 10) || 3000;
const dev = process.env.NODE_ENV !== "production";
const app = next({ dev });
const handle = app.getRequestHandler();

app
  .prepare()
  .then(() => {
    const server = express();

    server.get("/", (req, res) => {
      return app.render(req, res, "/", req.params);
    });

    server.get("/about", (req, res) => {
      return app.render(req, res, "/about", req.params);
    });

    server.get("*", (req, res) => {
      return handle(req, res);
    });

    server.listen(port, err => {
      if (err) throw err;
      console.log(`> Ready on http://localhost:${port}`);
    });
  })
  .catch(ex => {
    console.log(ex);
    process.exit(1);
  });
```

## update `package.json` with custom `server.js`

```json
{
  "scripts": {
    "dev": "node server.js",
    "build": "next build",
    "start": "NODE_ENV=production node server.js"
  }
}
```

## Install Apex Up

```shell
$ curl -sf https://up.apex.sh/install | sh

# verify the installation
$ up version
```

## AWS credentials

You need to have one AWS account and recommend to use IAM with programmaic way for security and convinience. If you have already installed `awscli` or `awsebcli`, etc. You are having `~/.aws/credentials` which is storing AWS credentials. It allows you to use `AWS_PROFILE` environment. If you don't please make one and save it with your account `access key` and `security key` in it.

```shell
# ~/.aws/credentials

[my-aws-account-for-lambda]
aws_access_key = xxxxxxxxxxxxx
aws_secret_access_key = xxxxxxxxxxxxxxxxxxxx
```

## IAM policy for Up CLI

IAM policy allows the Up to access your AWS resources in order to deploy your Next.js app on Lambda. Go to [AWS IAM](https://aws.amazon.com/iam/) and make the new policy and link it up to your AWS account.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "acm:*",
        "cloudformation:Create*",
        "cloudformation:Delete*",
        "cloudformation:Describe*",
        "cloudformation:ExecuteChangeSet",
        "cloudformation:Update*",
        "cloudfront:*",
        "cloudwatch:*",
        "ec2:*",
        "ecs:*",
        "events:*",
        "iam:AttachRolePolicy",
        "iam:CreatePolicy",
        "iam:CreateRole",
        "iam:DeleteRole",
        "iam:DeleteRolePolicy",
        "iam:GetRole",
        "iam:PassRole",
        "iam:PutRolePolicy",
        "lambda:AddPermission",
        "lambda:Create*",
        "lambda:Delete*",
        "lambda:Get*",
        "lambda:InvokeFunction",
        "lambda:List*",
        "lambda:RemovePermission",
        "lambda:Update*",
        "logs:Create*",
        "logs:Describe*",
        "logs:FilterLogEvents",
        "logs:Put*",
        "logs:Test*",
        "route53:*",
        "route53domains:*",
        "s3:*",
        "ssm:*",
        "sns:*"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "apigateway:*",
      "Resource": "arn:aws:apigateway:*::/*"
    }
  ]
}
```

## Create `up.json` file

```js
{
  "name": "nextjs-example",
  // aws account profile in ~/.aws/credentials
  "profile": "my-aws-account-for-lambda",
  "regions": ["ap-northeast-2"],
  "lambda": {
    // min 128, default 512
    "memory": 256,
    // AWS Lambda supports node.js 8.10 latest
    "runtime": "nodejs8.10"
  },
  "proxy": {
    "command": "npm start",
    "timeout": 25,
    "listen_timeout": 15,
    "shutdown_timeout": 15
  },
  "stages": {
    "development": {
      "proxy": {
        "command": "yarn dev"
      }
    }
  },
  "environment": {
    // you can hydrate env variables as you want.
    "NODE_ENV": "production"
  },
  "error_pages": {
    "variables": {
      "support_email": "admin@my-email.com",
      "color": "#2986e2"
    }
  }
}
```

## Build the Next.js app before deploy

```shell
$ yarn build
```

## Create `.upignore` file

The Up will inspect your files to compose and deploy to lambda. Firstly The up will read `.gitignore` and ignore files written in `.gitignore`. And after that, `.upignore` will be read. The Up, by default, ignores dotfiles, so needs to negate `.next` directory in `.upignore` in order for the Up will build the package with it.

```
# .upignore

!.next
```

## Deploy

```shell
$ up
```
