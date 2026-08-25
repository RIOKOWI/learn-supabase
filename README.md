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

# Read, Insert or Delete
- query for insert data :
```sql
INSERT INTO orders (address, zip_code, city, name, id, client_id, created_at)
VALUES 
('evangelion', 15560, 'Tangerang', 'Achyar', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Mawar No. 12', 15561, 'Tangerang', 'Budi', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Melati No. 8', 15562, 'Tangerang', 'Citra', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Kenanga No. 21', 15563, 'Tangerang', 'Dimas', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Anggrek No. 5', 15564, 'Tangerang', 'Eka', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Dahlia No. 17', 15565, 'Tangerang', 'Fajar', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Flamboyan No. 9', 15566, 'Tangerang', 'Gilang', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Cempaka No. 14', 15567, 'Tangerang', 'Hana', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Teratai No. 3', 15568, 'Tangerang', 'Indra', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Bougenville No. 25', 15569, 'Tangerang', 'Joko', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Sakura No. 11', 15570, 'Tangerang', 'Karin', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Kamboja No. 19', 15571, 'Tangerang', 'Lukman', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Tulip No. 7', 15572, 'Tangerang', 'Maya', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Lavender No. 16', 15573, 'Tangerang', 'Nanda', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Azalea No. 4', 15574, 'Tangerang', 'Oscar', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Gardenia No. 23', 15575, 'Tangerang', 'Putri', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Wijaya No. 10', 15576, 'Tangerang', 'Rizky', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Cemara No. 18', 15577, 'Tangerang', 'Salsa', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Pinus No. 6', 15578, 'Tangerang', 'Taufik', uuid_generate_v4(), uuid_generate_v4(), NOW()),
('Jl. Akasia No. 15', 15579, 'Tangerang', 'Vina', uuid_generate_v4(), uuid_generate_v4(), NOW());
```

- set new policy for user authenticated read data :
```sql
create policy "Enable read access for authenticated users"
on "public"."orders"
as PERMISSIVE
for SELECT
to authenticated
using (
  true
);
```

- example code for read data on console :

```js
<script setup lang="ts">
import { ref } from "vue"
import { supabase } from '@/supabase';
import User from "@/components/User.vue"

const pagination = ref(9)

const fetchOrders = async () => {

  try {
    let { data: orders, error } = await supabase
    .from('orders')
    .select('*') // you can show selected field, example .select('id, name`)
    .range(0, pagination.value) // you can add pagination, example show data from index 1-9
    if(orders){
      console.log(orders)
    }
  } catch (error) {
    console.log(error)
  }
}

fetchOrders();
</script>

<template>
  <main class="flex items-center justify-between">
   
  </main>
</template>
```

- data :
```log
0: 
  id: 
  "04bf8012-e205-44f0-8520-d521dc88aadd"
  name: 
  "Sup Sup"
```