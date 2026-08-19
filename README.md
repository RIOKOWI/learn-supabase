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

- example of Enable insert for authenticated users only rls : 
```sql
alter policy "Enable insert for authenticated users only"
on "public"."clients"
to authenticated
with check (
  true
);
```