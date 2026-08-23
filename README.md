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