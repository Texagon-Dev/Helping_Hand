This will Create the Customer Dependent on the Auth Table. The User in customer will be Created or Deleted when its entry is hit in auth table. Modify this Code According to your Needs 
````

create  table
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

CREATE
OR REPLACE FUNCTION public.handle_user_update () RETURNS TRIGGER LANGUAGE plpgsql SECURITY DEFINER
SET
  search_path = public AS $$
begin
  update public."customer"
  set metadata = new.raw_user_meta_data
  where uuid = new.id;
  return new;
end;
$$;

CREATE
OR REPLACE TRIGGER on_auth_user_updated
AFTER
UPDATE ON auth.users FOR EACH ROW
EXECUTE PROCEDURE public.handle_user_update ();
````


In Order to Drop these , Use These Commands 
````
-- Drop triggers first
DROP TRIGGER IF EXISTS on_auth_user_updated ON auth.users;
DROP TRIGGER IF EXISTS on_auth_user_created ON auth.users;

-- Drop functions
DROP FUNCTION IF EXISTS public.handle_user_update();
DROP FUNCTION IF EXISTS public.handle_new_user();

-- Drop indexes
DROP INDEX IF EXISTS public.idx_customer_plan_plan_id;
DROP INDEX IF EXISTS public.idx_customer_plan_customer_uuid;

-- Drop tables
-- Note: Order is important due to foreign key constraints
DROP TABLE IF EXISTS public.customer_plan;
DROP TABLE IF EXISTS public.plans;
DROP TABLE IF EXISTS public.customer;
````
