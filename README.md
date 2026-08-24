# Docs
[DOCUMENTATION PAGE](https://supabase.com/docs)


# Tables
### ENPOINT API`s
>> Base Url on .env file (not comitted)

ENDPOINT CLIENT
```
{{BASE_URL}}clients 
{{BASE_URL}}orders
```

# Row Level Security
- anon = untuk semua orang
* note : memberi rls bisa pakai template yang di sediakan supabase
---
- example of Enable insert for authenticated users only rls : 
```sql
alter policy "Enable insert for authenticated users only"
on "public"."clients"
to authenticated
with check (
  true
);
```
---
- policy untuk memperbolehkan user update based on email
```sql
alter policy "enable update for user by email"
on "public"."clients"
to public
using (
  ((auth.jwt() ->> 'email'::text) = email)
) with check (
  ((auth.jwt() ->> 'email'::text) = email)
);
```
---
- Enable select for clients based on user_id == user yang sudah login hanya bisa lihat punya dia sendiri
```sql
alter policy "Enable select for clients based on user_id"
on "public"."clients"
to authenticated
using (
  (( SELECT auth.uid() AS uid) = user_id)
);
```

# Authentication
you need `anon key`, `base url` and `callback url` for authentication


# User Management

```js
<script setup>
import { supabase } from "@/supabase"
import { useAuth } from "@/composables/useAuth"

const { router, setUser } = useAuth()

const signOut = async () => {
  let { error } = await supabase.auth.signOut()
  setUser(null)
  await router.push('/')
}
</script>

<template>
  <header class="py-4 bg-gray-900">
    <div class="container flex items-center justify-between mx-auto">
      <h1 class="text-lg font-bold text-white">My Nuxt Supabase app</h1>
      <button class="w-auto px-4 py-2 font-semibold text-white bg-red-600 rounded hover:bg-red-700" @click="signOut">
        Signout
      </button>
    </div>
  </header>
</template>
```


# Recover Password
- you need recovery password page
- you need url callback `http://localhost:3000/profile/update` for update profile
- rate limit 2 email/hour
- example code of recovery pass page :
```js
<script setup>
import Alert from "@/components/Alert.vue"
import { reactive } from 'vue'
import { useAuth } from "@/composables/useAuth"
import { supabase } from '@/supabase'

const state = reactive({
  loading: false,
  success: false,
  email: undefined,
})

const { validateEmail } = useAuth()

const recoverPassword = async () => {
  state.error = undefined;
  if (!validateEmail(state.email)) {
    state.error = "Enter a valid email."
    return
  }
  try {
    state.loading = true
    let { data, error } = await supabase
      .auth
      .resetPasswordForEmail(state.email)

    if (data) {
      state.success = "Successfully reset password."
    }
  } catch (error) {
    state.error = "Error coming from Supabase."
  } finally {
    state.loading = false
  }
}
</script>

<template>
  <div class="container mx-auto text-center w-96">
    <header class="my-6">
      <h1 class="mb-2 text-2xl font-bold">Recover your password</h1>
      <p>You'll receive an email to recover your password.</p>
    </header>
    <div class="mx-auto">
      <input type="text" class="text" placeholder="mail@gmail.com" v-model="state.email" />
    </div>
    <Alert v-if="state.error" :text="state.error" class="alert-danger" />
    <Alert v-if="state.success" :text="state.success" class="alert-success" />
    <div>
      <button class="btn btn-primary" :disabled="state.loading" @click="recoverPassword">{{ state.loading ? 'Loading' :
        'Send' }}</button>
    </div>
  </div>
</template>

<style lang='scss'></style>
```

# Email Templates
- you can styling email templates supabase by default
- examle code :
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>You've been invited</title>
</head>

<body style="margin:0; padding:0; background-color:#f4f4f5; font-family:Arial, Helvetica, sans-serif; color:#18181b;">

  <table width="100%" cellpadding="0" cellspacing="0" border="0" style="background-color:#f4f4f5; padding:40px 20px;">
    <tr>
      <td align="center">

        <!-- Container -->
        <table width="100%" cellpadding="0" cellspacing="0" border="0" style="max-width:520px;">

          <!-- Logo / Brand -->
          <tr>
            <td align="center" style="padding-bottom:24px;">
              <div style="
                display:inline-block;
                background-color:#111111;
                color:#ffffff;
                font-size:20px;
                font-weight:bold;
                padding:12px 18px;
                border-radius:10px;
                letter-spacing:-0.5px;
              ">
                LEARN SUPABASE
              </div>
            </td>
          </tr>

          <!-- Card -->
          <tr>
            <td style="
              background-color:#ffffff;
              border:1px solid #e4e4e7;
              border-radius:16px;
              padding:40px 36px;
            ">

              <h1 style="
                margin:0 0 16px;
                font-size:28px;
                line-height:36px;
                font-weight:700;
                color:#18181b;
              ">
                You've been invited by Rio Achyar
              </h1>

              <p style="
                margin:0 0 28px;
                font-size:16px;
                line-height:26px;
                color:#52525b;
              ">
                You've been invited to create an account on
                <strong style="color:#18181b;">LEARN SUPABASE</strong>.
                Click the button below to accept your invitation and get started.
              </p>

              <!-- CTA -->
              <table cellpadding="0" cellspacing="0" border="0">
                <tr>
                  <td align="center" style="border-radius:10px; background-color:#111111;">
                    <a
                      href="{{ .ConfirmationURL }}"
                      style="
                        display:inline-block;
                        padding:14px 24px;
                        font-size:15px;
                        font-weight:600;
                        color:#ffffff;
                        text-decoration:none;
                        border-radius:10px;
                      "
                    >
                      Accept Invitation
                    </a>
                  </td>
                </tr>
              </table>

              <!-- Divider -->
              <div style="
                height:1px;
                background-color:#e4e4e7;
                margin:32px 0 24px;
              "></div>

              <p style="
                margin:0 0 8px;
                font-size:13px;
                line-height:20px;
                color:#71717a;
              ">
                If the button above doesn't work, copy and paste this link into your browser:
              </p>

              <p style="
                margin:0;
                font-size:12px;
                line-height:20px;
                word-break:break-all;
              ">
                <a
                  href="{{ .ConfirmationURL }}"
                  style="color:#2563eb; text-decoration:none;"
                >
                  {{ .ConfirmationURL }}
                </a>
              </p>

            </td>
          </tr>

          <!-- Footer -->
          <tr>
            <td align="center" style="padding:24px 20px;">

              <p style="
                margin:0 0 8px;
                font-size:12px;
                line-height:18px;
                color:#a1a1aa;
              ">
                This invitation was sent automatically.
              </p>

              <p style="
                margin:0;
                font-size:12px;
                line-height:18px;
                color:#a1a1aa;
              ">
                © 2026 LEARN SUPABASE. All rights reserved.
              </p>

            </td>
          </tr>

        </table>

      </td>
    </tr>
  </table>

</body>
</html>
```

# URL Configuration
`http://localhost:3000` -> Development
`http://bonjour` -> Production

* Note : change the basic url for production apps/web

- feature Redirect URL`s its the same use case for redirect to page to another page