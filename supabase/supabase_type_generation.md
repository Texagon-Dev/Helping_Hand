# Supabase Type Generation

Use this script to generate type for supabase without an ORM:

```
npx supabase gen types --lang=typescript --project-id ${PROJECT_ID} --schema ${SCHEMA_NAME} > ${OUTPUT_PATH}/${FILE_NAME}.types.ts
```
This will create a file that exports the Database type.The Database type looks like this:
```
export interface Database {
  public: {
    Tables: {
      movies: {
        Row: {
          // the data expected from .select()
          id: number
          name: string
          data: Json | null
        }
        Insert: {
          // the data to be passed to .insert()
          id?: never // generated columns must not be supplied
          name: string // `not null` columns with no default must be supplied
          data?: Json | null // nullable columns can be omitted
        }
        Update: {
          // the data to be passed to .update()
          id?: never
          name?: string // `not null` columns are optional on .update()
          data?: Json | null
        }
      }
    }
  }
}
```
Use this type when creating the Supabase Client like this to enable autocompletion, API definitions, RPC definitions etc:
```
const supabase = createClient<Database>(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_KEY
);
```
Docs: https://supabase.com/docs/guides/api/rest/generating-types