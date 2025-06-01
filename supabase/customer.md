This will Create the user Dependent on the Auth Table. The User in user will be Created or Deleted when its entry is hit in auth table. Modify this Code According to your Needs 
````

create  table
  public.user (
    created_at timestamp with time zone not null default now(),
    uuid uuid not null,
    email character varying not null,
    metadata jsonb null default '[]'::jsonb,
    constraint user_pkey primary key (uuid),
    constraint user_uuid_fkey foreign key (uuid) references auth.users (id) on delete cascade
  ) tablespace pg_default;

create or replace function public.handle_new_user()
returns trigger
language plpgsql
security definer
set search_path = public
as $$
begin
  insert into public."user" ("uuid", "email", "metadata")
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
  update public."user"
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
DROP INDEX IF EXISTS public.idx_user_plan_plan_id;
DROP INDEX IF EXISTS public.idx_user_plan_user_uuid;

-- Drop tables
-- Note: Order is important due to foreign key constraints
DROP TABLE IF EXISTS public.user_plan;
DROP TABLE IF EXISTS public.plans;
DROP TABLE IF EXISTS public.user;
````
