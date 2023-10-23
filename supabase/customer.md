--Customer is Dependent on Auth User Completely
````
create table
  public.customer (
    created_at timestamp with time zone not null default now(),
    uuid uuid not null,
    email character varying not null,
    metadata jsonb null default '[]'::jsonb,
    constraint customer_pkey primary key (uuid),
    constraint customer_uuid_fkey foreign key (uuid) references auth.users (id) on delete cascade
  ) tablespace pg_default;

create or replace function public.handle_new_user()
returns trigger
language plpgsql
security definer
set search_path = public
as $$
begin
  insert into public."customer" ("uuid", "email", "metadata")
  values (new.id, new.email, new.raw_user_meta_data);
  return new;
end;
$$;

create or replace trigger on_auth_user_created
after insert on auth.users
for each row
execute procedure public.handle_new_user();
````
