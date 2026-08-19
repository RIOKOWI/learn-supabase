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