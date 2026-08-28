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
    .eq('name', 'Joko') // untuk filter data by name
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

- example code of insert data :
```js
<script setup lang="ts">
import { ref } from "vue"
import { supabase } from '@/supabase';
import User from "@/components/User.vue"

const pagination = ref(9)


const insertOrder = async () => {
  try {
    
  let { data, error } = await supabase
    .from('orders')
    .insert({
      address: 'badung',
      zip_code: '192898',
      city: 'Hungar',
      name: 'wiwi',
      client_id: '07ef2e70-5790-4449-8b96-b272127a1799',
      price: 69
    })
    .select()
  if(data){
    console.log(data)
  }
  } catch (error) {

  }
}

const fetchOrders = async () => {

  try {
    let { data: orders, error } = await supabase
    .from('orders')
    .select('*')
    .eq('name', 'Joko')

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
    <div class="px-2 my-8">
      <button class="block btn btn-primary" @click="insertOrder">+ Insert</button>
    </div>
  </main>
</template>
```

- rls to delete :
```sql
create policy "Enable delete for users based on user_id"
on "public"."orders"
as PERMISSIVE
for DELETE
to authenticated
using (
  (select auth.uid()) = user_id
);
```


# Subscription
- Realtime Changes
- example code subcription:
```js
<script setup lang="ts">
import { ref } from "vue"
import { supabase } from '@/supabase';
import User from "@/components/User.vue"


interface Order {
  name: string,
  id: string,
  created_at: string,
  client_id: string,
  price: number,
  address: string,
  city: string,
  zip_code: string,
}

const orders = ref<Order[]>([])

const pagination = ref(9)



// const insertOrder = async () => {
//   try {
    
//   let { data, error } = await supabase
//     .from('orders')
//     .insert({
//       address: 'badung',
//       zip_code: '192898',
//       city: 'Hungar',
//       name: 'wiwi',
//       client_id: '07ef2e70-5790-4449-8b96-b272127a1799',
//       price: 69
//     })
//     .select()
//   if(data){
//     console.log(data)
//   }
//   } catch (error) {

//   }
// }

const fetchOrders = async () => {

   try {
    const { data, error } = await supabase
      .from('orders')
      .select('*')
    if (data) {
      console.log(data)
      orders.value = data
    }
  } catch (error) {
    console.log(error)
  }
}

// const updateWiwi = async () => {
//   try {
//     let { data, error } = await supabase
//       .from('orders')
//       .update({ price: 2000 })
//       .eq('name', 'Hana')
//       .select()

//       if(data){
//           console.log(data)
//         }
//   } catch (error) {
//     console.log(error)
//   }
// }


// const deleteWiwi = async () => {
//     let { error } = await supabase
//       .from('orders')
//       .delete()
//       .eq('name', 'Fajar')
// }

fetchOrders();

const channel = supabase
  .channel('my_new_channel_order')
  .on(
    'postgres_changes',
    {
      event: '*',
      schema: 'public',
      table: 'orders',
      // filter: 'body=eq' bisa kasih filter seperti di contoh
    },
    (event) => {
      console.log(event) // show change on the log
      // const { new: newOrder } = event;  showing realtime changes
      // orders.value = orders.value.map(order => {
      //   if (order.id === newOrder.id) {
      //     return {
      //       ...order,
      //       ...newOrder
      //     }
      //   }
      //   return order
      // })
    }
  )

.subscribe()
</script>

<template>
  <main class="flex items-center justify-between">
    <div class="container px-2 mx-auto my-8">
      <User />
      <div v-for="order, index in orders" :key="index">
        <div v-if="order" class="grid grid-cols-5 gap-3 px-3 py-2 my-2 border rounded-lg shadow-md">
          <div v-if="order.name">{{ order.name }}</div>
          <div>${{ order.price || 0 }}</div>
          <div v-if="order.address">{{ order.address }}</div>
          <div v-if="order.zip_code">{{ order.zip_code }}</div>
          <div v-if="order.city">{{ order.city }}</div>
        </div>
      </div>
    </div>
  </main>
</template>
```

- console result
```log
client.ts:19 [vite] connecting...
client.ts:134 [vite] connected.
Home.vue:53 (19) [{…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}]0: {id: '5a7baf43-df7e-4fe8-bb0a-757c71b10ff3', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '995045dc-0d22-4355-95d4-b594e325a60d', price: null, address: 'evangelion', …}1: {id: 'd5c7c9a5-baa2-430d-a793-9a272719e503', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: 'f6cae42b-bd9a-4a7c-8460-d9ddf07fa005', price: null, address: 'Jl. Mawar No. 12', …}2: {id: '1bb7c532-4553-4bbf-921e-a4afe2083b25', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '69ffea01-fd38-4885-8fc1-5485c920a9fb', price: null, address: 'Jl. Melati No. 8', …}3: {id: 'd3358022-2605-4120-bf29-8c7c18aa80ae', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '42175f9f-1660-41d3-8285-a5bd42f821fc', price: null, address: 'Jl. Kenanga No. 21', …}4: {id: '46308b12-792f-409e-a3d1-3c3a85d4f34d', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '732a2c4a-1b6a-40f3-be49-f63d416af390', price: null, address: 'Jl. Anggrek No. 5', …}5: {id: '2ea80d5a-06d1-4179-8b3e-c53517f4c4db', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '6cbb044a-29e2-4145-b924-d8d6365f998e', price: null, address: 'Jl. Flamboyan No. 9', …}6: {id: '8076cc02-3dc0-4038-94a4-9237c2e3e847', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: 'e1bc49c5-4692-42e0-9b48-210afdbe1842', price: null, address: 'Jl. Cempaka No. 14', …}7: {id: '9398e277-ed9f-431c-8af9-45606ecc5435', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '4ff7ff81-2ee9-4ae2-a476-04c0ab214f6f', price: null, address: 'Jl. Teratai No. 3', …}8: {id: '06ad6ea5-440f-4c15-be33-009af40005c6', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '4d683301-d5e7-4d0c-a846-756d4e1be5f3', price: null, address: 'Jl. Bougenville No. 25', …}9: {id: '470c0cf1-ac7b-4eb7-b0fa-cdd58a8d0ff6', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '24be21ef-fd81-4e97-a5cc-62e64edcfeb8', price: null, address: 'Jl. Sakura No. 11', …}10: {id: '54ad4395-301b-4d0f-8d3d-66efb8c3da41', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: 'c98d54b9-f761-4025-88d5-2f14713256cb', price: null, address: 'Jl. Kamboja No. 19', …}11: {id: 'a3b1eff0-043d-493b-ba3f-a516ecdb5d2a', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '17bf25c5-5cdb-4252-9407-62f5f3b45138', price: null, address: 'Jl. Tulip No. 7', …}12: {id: '1faaa96e-08b6-4095-8af2-df5be4a74526', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '54079747-0d5c-4726-a055-f5417fdddd9b', price: null, address: 'Jl. Lavender No. 16', …}13: {id: '88277337-dfa2-45c7-b353-f99f98bdb57f', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '899c46a8-53af-43f6-bb2e-f38041aa9325', price: null, address: 'Jl. Azalea No. 4', …}14: {id: 'ad442508-4c38-441f-9b92-bbfe3cfaa5ce', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: 'a6a5c567-3629-405d-b086-32b0c55a8201', price: null, address: 'Jl. Gardenia No. 23', …}15: {id: '3fa1df38-9b59-4d66-8707-4ea4e212456d', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '6be6ab4a-008e-453e-b83f-a53010411346', price: null, address: 'Jl. Cemara No. 18', …}16: {id: '791a4201-ffc4-4125-9324-7b33e89f27d9', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: '3c0aa2f9-2fbe-4744-b003-98001685dfd5', price: null, address: 'Jl. Pinus No. 6', …}17: {id: 'd4a63e46-3fc7-4e07-bf19-892c4959d34d', created_at: '2026-08-25T11:41:39.312768+00:00', client_id: 'f65f889c-c8bd-4a36-96ca-de9837912c22', price: null, address: 'Jl. Akasia No. 15', …}18: {id: '04bf8012-e205-44f0-8520-d521dc88aadd', created_at: '2026-08-19T07:28:54.720354+00:00', client_id: '07ef2e70-5790-4449-8b96-b272127a1688', price: 20, address: 'Jl. Embut', …}length: 19[[Prototype]]: Array(0)
Home.vue:98 {schema: 'public', table: 'orders', commit_timestamp: '2026-08-26T12:37:47.551Z', eventType: 'UPDATE', new: {…}, …}commit_timestamp: "2026-08-26T12:37:47.551Z"errors: nulleventType: "UPDATE"new: address: "Jl. Embut Embut"city: "Tarakarta"client_id: "07ef2e70-5790-4449-8b96-b272127a1688"created_at: "2026-08-19T07:28:54.720354+00:00"id: "04bf8012-e205-44f0-8520-d521dc88aadd"name: "Sup Sup"price: 20zip_code: "232352"[[Prototype]]: Objectold: id: "04bf8012-e205-44f0-8520-d521dc88aadd"[[Prototype]]: Objectschema: "public"table: "orders"[[Prototype]]: Objectconstructor: ƒ Object()hasOwnProperty: ƒ hasOwnProperty()isPrototypeOf: ƒ isPrototypeOf()propertyIsEnumerable: ƒ propertyIsEnumerable()toLocaleString: ƒ toLocaleString()toString: ƒ toString()valueOf: ƒ valueOf()__defineGetter__: ƒ __defineGetter__()__defineSetter__: ƒ __defineSetter__()__lookupGetter__: ƒ __lookupGetter__()__lookupSetter__: ƒ __lookupSetter__()__proto__: (...)get __proto__: ƒ __proto__()set __proto__: ƒ __proto__()

```

# Relations
- you can add on the Database->Tables without query

# Function
- Function = tempat menyimpan logic yang ingin dijalankan di database.
- example query create function :
```sql
create or replace function increment (row_id uuid)
returns void as
$$
  update orders
  set view = view + 1
  where row_id = id;
$$
language sql volatile;
```

- meaningful comments of query function
```sql
-- Membuat function bernama "increment" atau mengganti function
-- yang sudah ada dengan nama dan parameter yang sama.
create or replace function increment (row_id uuid)

-- Function tidak mengembalikan nilai apa pun.
returns void as

$$
  -- Menaikkan nilai kolom "view" pada tabel "orders" sebesar 1.
  update orders

  -- Menambahkan 1 ke jumlah view yang sudah ada.
  set view = view + 1

  -- Hanya mengubah data order yang memiliki ID sesuai dengan
  -- UUID yang diberikan melalui parameter "row_id".
  where row_id = id;
$$

-- Menentukan bahwa function menggunakan SQL sebagai bahasanya.
language sql

-- Menandakan bahwa function dapat menghasilkan efek samping
-- seperti melakukan UPDATE dan hasilnya tidak selalu sama.
volatile;
```

- example function code :
```js
const incrementViews = async () => {
  try {
    const { data, error } = await supabase.rpc('increment', {
      row_id: '04bf8012-e205-44f0-8520-d521dc88aadd'
    })

    if (data) {
      console.log("success")
      console.log(data)
    }

  } catch (error) {
    console.log(error)
  }
}
```


# Triggers
- trigger is a function that run before of after
- example query :
```sql
CREATE TABLE USERS (
  id uuid references auth.users not null primary key,
  email text
);

CREATE OR replace function public.handle_new_user()
returns trigger as $$
begin
  insert into public.users (id, email)
  values (new.id, new.email);
  return new;
end;
$$ language plpgsql security definer;

create trigger on_new_user
  after insert on auth.users
  for each row execute procedure public.handle_new_user()
```

- query with meaningfull comments :
```sql
-- Membuat tabel "users" untuk menyimpan data tambahan pengguna.
-- ID pengguna diambil dari auth.users milik Supabase Auth.
CREATE TABLE USERS (

  -- ID pengguna dari Supabase Auth.
  -- Menjadi primary key dan harus memiliki user yang sesuai di auth.users.
  id uuid references auth.users not null primary key,

  -- Menyimpan alamat email pengguna.
  email text

);


-- Membuat atau mengganti function yang akan dijalankan
-- secara otomatis ketika ada user baru di Supabase Auth.
CREATE OR replace function public.handle_new_user()

-- Karena function ini dipanggil oleh trigger,
-- function harus mengembalikan data bertipe "trigger".
returns trigger as $$

begin

  -- Menambahkan user baru ke tabel public.users.
  insert into public.users (id, email)

  -- Mengambil ID dan email dari user yang baru dibuat
  -- di tabel auth.users.
  values (new.id, new.email);

  -- Mengembalikan data user yang baru dibuat
  -- agar proses trigger dapat dilanjutkan.
  return new;

end;

$$ language plpgsql security definer;


-- Membuat trigger yang otomatis berjalan
-- setelah user baru berhasil dibuat di auth.users.
create trigger on_new_user

  -- Trigger dijalankan setelah proses INSERT selesai.
  after insert on auth.users

  -- Trigger dijalankan untuk setiap user baru yang dibuat.
  for each row execute procedure public.handle_new_user();
```


# Schemas
- wadah untuk mengelompokkan tabel
- contoh : public.orders  | public = schema & orders = table
- exmample query create schemas :
```sql
CREATE SCHEMA sales;

CREATE TABLE sales.customer (
  id uuid not null primary key,
  name text,
  email text
);
```

# Storage 
- tempat untuk nyimpan/upload pho,vid,file, dll
- free plan only 1 gb
- after you create a bucket you must set row level securiy/policies for tour bucket
```sql
create policy objects_select_policy on storage.objects for select
  using (auth.role() = 'authenticated');

create policy object_insert_policy on storage.objects for insert
  with check (auth.role() = 'authenticated');

create policy objects_update_policy on storage.objects for update
  using (auth.role() = 'authenticated');

create policy objects_delete_policy on storage.objects for delete
  using (auth.role() = 'authenticated');
```

- query with meaningfull comment
```sql
-- Allow authenticated users to view files stored in Supabase Storage.
create policy objects_select_policy
on storage.objects
for select
using (auth.role() = 'authenticated');


-- Allow authenticated users to upload new files to Supabase Storage.
create policy object_insert_policy
on storage.objects
for insert
with check (auth.role() = 'authenticated');


-- Allow authenticated users to update existing files in Supabase Storage.
create policy objects_update_policy
on storage.objects
for update
using (auth.role() = 'authenticated');


-- Allow authenticated users to delete files from Supabase Storage.
create policy objects_delete_policy
on storage.objects
for delete
using (auth.role() = 'authenticated');
```

- example implement code 
```js
<script>
import { ref } from 'vue'
import { supabase } from '@/supabase';

export default {
  name: 'AvatarUploader',
  setup() {
    const fileInput = ref(null)
    const imageUrl = ref(null)
    const showOverlay = ref(false)

    const handleFileInputChange = async (event) => {
      const file = event.target.files[0]
      if (!file) return

      showOverlay.value = true
      const { data, error } = await supabase
        .storage
        .from('avatars')
        .upload(`avatar_${Date.now()}.png`, file)
      showOverlay.value = false
      if (error) {
        console.error(error)
        return
      }
      imageUrl.value = URL.createObjectURL(file)
    }

    return {
      fileInput,
      imageUrl,
      showOverlay,
      handleFileInputChange,
    }
  },
}
</script>

<template>
  <div class="relative w-24 h-24 overflow-hidden border-2 border-gray-300 rounded-full cursor-pointer">
    <img v-if="imageUrl" :src="imageUrl" class="object-cover w-full h-full">
    <div v-else class="flex items-center justify-center w-full h-full text-gray-500 cursor-pointer">
      <svg xmlns="http://www.w3.org/2000/svg" class="w-8 h-8" viewBox="0 0 20 20" fill="currentColor">
        <path fill-rule="evenodd" d="M10 0a10 10 0 1 0 10 10A10 10 0 0 0 10 0zM9 9H5v2h4v4h2v-4h4V9h-4V5H9v4z"
          clip-rule="evenodd" />
      </svg>
    </div>
    <label for="avatar-upload" class="absolute inset-0 opacity-0 cursor-pointer">
      <span class="sr-only">Upload avatar</span>
      <input id="avatar-upload" type="file" @change="handleFileInputChange" class="w-full h-full">
    </label>
    <div v-if="showOverlay" class="absolute inset-0 flex items-center justify-center bg-gray-500 opacity-50 cursor-pointer">
      <svg xmlns="http://www.w3.org/2000/svg" class="w-8 h-8 text-white animate-pulse" viewBox="0 0 20 20"
        fill="currentColor">
        <path fill-rule="evenodd" d="M10 0a10 10 0 1 0 10 10A10 10 0 0 0 10 0zM9 9H5v2h4v4h2v-4h4V9h-4V5H9v4z"
          clip-rule="evenodd" />
      </svg>
    </div>
  </div>
</template>
```