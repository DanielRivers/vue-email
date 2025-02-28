---
title: NodeMailer
description: Learn how to send an email using Vue Email and Nodemailer.
links:
  - label: NPM
    icon: i-simple-icons-npm
    to: https://www.npmjs.com/package/nodemailer
---

## 1. Install dependencies

Get the [nodemailer](https://www.npmjs.com/package/nodemailer) package.

::code-group
```sh [pnpm]
pnpm add nodemailer
```
```sh [yarn]
yarn add nodemailer
```
```sh [npm]
npm install nodemailer
```
::

## 2. Create an email using Vue

Start by building your email template in a `.vue` file.


```vue [emails/welcome.vue]
<script lang="ts" setup>
defineProps<{ url: string }>();
</script>

<template>
  <e-html lang="en">
    <e-button :href="url">
      View on GitHub
    </e-button>
  </e-html>
</template>
```

## Step 3: Convert to HTML and send email

Import the email template you just built, convert into a HTML string, and use the Nodemailer SDK to send it.

::code-group

```ts [Nuxt 3]
// server/api/send-email.post.ts
import { useCompiler } from '#vue-email'
import nodemailer from 'nodemailer';

export default defineEventHandler(async (event) => {
  const template = await useCompiler('welcome.vue', {
    props: {
      url: 'https://vuemail.net/',
    }
  })

  const testAccount = await nodemailer.createTestAccount();

  const transporter = nodemailer.createTransport({
    host: process.env.HOST || 'smtp.ethereal.email',
    port: 587,
    secure: false,
    auth: {
      user: testAccount.user,
      pass: testAccount.pass,
    },
  });

  const options = {
    from: 'you@example.com',
    to: 'user@gmail.com',
    subject: 'hello world',
    html: template,
  };

  await transporter.sendMail(options);
  return { message: 'Email sent' };
});
```

```ts [NodeJs]
import express from 'express';
import nodemailer from 'nodemailer';
import { config } from "vue-email/compiler";

const app = express();
const vueEmail = config("./emails");

app.use(express.json());

app.post('/api/send-email', async (req, res) => {
  const template = await vueEmail.render("welcome.vue", {
      props: {
        url: 'https://vuemail.net/',
      },
    });

  const testAccount = await nodemailer.createTestAccount();

  const transporter = nodemailer.createTransport({
    host: process.env.HOST || 'smtp.ethereal.email',
    port: 587,
    secure: false,
    auth: {
      user: testAccount.user,
      pass: testAccount.pass,
    },
  });

  const options = {
    from: 'you@example.com',
    to: 'user@gmail.com',
    subject: 'hello world',
    html: template,
  };

  await transporter.sendMail(options);

  return res.json({ message: "Email sent" });
});

app.listen(3001);
```

::
