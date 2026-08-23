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