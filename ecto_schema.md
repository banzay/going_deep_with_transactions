
 Ecto schema is used to map any data source into an Elixir struct.
 The definition of the schema is possible through two main APIs: `schema/2` and `embedded_schema/1`
 `schema` - for database tables
 `embedded_schema` - for schemas, which are embedded in other schemas, or exist only in memory (not in db).

 `schema` does:
 - autogenerate `:integer` primary key
 - mostly used as parameter for `Ecto.Changeset` methods, but can be used as a regular struct.

 ### schema
 `@primary_key` \ {:id, :id, autogenerate: true}
 `@timestamps_opts` \ [type: :naive_datetime]

Overwrite to use `UUID`:
 ```elixir
defmodule MyApp.Schema do
  defmacro __using__(_) do
    quote do
      use Ecto.Schema
      @primary_key 
        {:id, :binary_id, autogenerate: true}
      @foreign_key_type :binary_id
    end
  end
end
 ```
Fields in schema can be of type:
1. Built-in type:
    - binary `:binary_id` 
    - UTF-8 encoded `:string`
    - integer `:id`, `:integer` 
    - float `:float` 
    - Decimal `:decimal` 
    - map `:map`, `{:map, inner_type}` 
    - list `{:array, inner_type}` 
    - Date `:date` 
    - Time `:time`, `:time_usec` 
    - NaiveDateTime `:naive_datetime`, `:naive_datetime_usec` 
    - DateTime `:utc_datetime`, `:utc_datetime_usec`

2. Custom type
    - Ecto.UUID `:uuid`

Map type in Postgres relies on JSON library:
`{:jason, "~> 1.0"}`

Schema reflection (autogenerated methods):
`__schema__(:primary_key)`, `__schema__(:fields)`, `__schema__(:type, field)`, `__schema__(:associations)`, `__schema__(:embeds)`...
`__struct__`, `__changeset__` - to manipulate as Struct or by Ecto.Changeset.
 
 
 ### Functions

embeds_one/3
embeds_one/4
embeds_many/3
embeds_many/4

```elixir
has_one(other_schema, queryable, options[
    foreign_key
        \ other_schema <> "_id"
    references 
        \ primary key
    through (NEVER USE IT - it's only readable. Usse writable many_to_many)
        # see has_many/has_one :through
    on_delete
        \ :nothing
        | :nilify_all (NEVER USE THIS)
        | :delete_all (NEVER USE THIS)
    # Prefer :nilify_all, :delete_all in migrations!

    on_replace
        \ :raise
        | :mark_as_invalid
        | :nilify
        | :update
        | :delete
    # TODO: See ecto_changeset.md:on_replace
    
    defaults
        any_term
        | {__MODULE__, :update_comment, []}
    # Used with Ecto.build_assoc(post, :comments)
    # Prefer default values in migrations!

    where (NEVER USE THIS)
        # TODO: See "Filtering associations" in has_many/3  
])
    
```

```elixir
has_many(name, queryable, opts \\ [
    ...same as in has_one
])

has_many :posts, Post
has_many :children, Comment, foreign_key: :parent_id, references: :id
```

###  has_many/has_one :through
Read only!

For writable polymorphic association - use  (see [many_to_many](ecto_polymorphism.md))

```elixir
# posts schema
has_many :comments_authors, through: [:comments, :author]

# comments schema
belongs_to :author, Author
belongs_to :post, Post

# Get associated authors
assoc(post, [:comments, :author]) |> Repo.all()

# Preload associated authors
from(p in Post, where: p.id == 42, preload: :comments_authors) |> Repo.all()
```



belongs_to/3

many_to_many/3

timestamps/1
 
 ### embedded_schema

