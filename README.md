# Fictitious

`Fictitious` is a tool that enables you to create a fictitious data or records. It helps you to create mock data for your unit test
without having the hassle of preparing the data in an convoluted order according to their associations that they have. `Fictitious`
will ensure that whatever ecto schema that you specified, you will get the schema's data created for you.

## Installation

`Fictitious` only generates fictitious data hence it is recommended that you install them in test environment
only but feel free to play around with `Fictitious` by having it installed in normal dev environment.
Inside your `mix.exs` file add `{:fictitious, "~> 0.1.1", only: :test}` as one of your dependency:

```elixir
defp deps do
  [
    ...
    {:fictitious, "~> 0.1.1", only: :test},
    ...
  ]
end
```

Once you have it installed, you need to configure the `Fictitious` repo. To do so add the following configuration
to your `test.exs` file:

```elixir
config :fictitious, :repo,
  default: YourApp.Repo
```

Notice we put it inside `test.exs` file. This is because `Fictitious` is engineered to help you in creating mock data for
your unit test hence most of the time you will definitely put this configuration in test environment only.

In case your application has more than one `Repo` it is possible to configure more than one repo for `Fictitious` to
use. In fact, you could configure as many as you want. To do so you could add more repos into the fictitious `:repo` configs
as follow:

```elixir
config :fictitious, :repo,
  default: YourApp.Repo,
  second_repo: YourApp.SecondRepo,
  third_repo: YourApp.ThirdRepo
```

The `default` repo will be used by default by `Fictitious`.

## How to Use

### Basics

Given you have the following ecto schema inside your app:

```elixir
defmodule YourApp.Schema.Person do
  use Ecto.Schema
  import Ecto.Changeset

  schema "persons" do
    field :name, :string
    field :age, :integer
    field :email, :string

    timestamps()
  end

  ...
end
```

To generate fictitious data of a person you could simply call `fictionize/1` function as follow:

```elixir
iex> Fictitious.fictionize(YourApp.Schema.Person)
{:ok, %YourApp.Schema.Person{
  __meta__: #Ecto.Schema.Metadata<:loaded, "persons">,
  id: 35,
  name: "2cBfxcqnB0B5iqhYvK83RamaDa8KM0PvPpT1kVao",
  age: 514,
  email: "2cBfxcqnB0B5iqhYvK83RamaDa8KM0PvPpT1kVao",
  inserted_at: ~U[2020-04-31 06:19:27Z],
  updated_at: ~U[2020-04-31 06:19:27Z]
}}
```
    
`fictionize/1` will simply generate fictitious value that is according to the field's type. The currently supported
primitive types by `Fictitious` can be found in [official ecto documentation](https://hexdocs.pm/ecto/Ecto.Schema.html#module-primitive-types).

In case you want some fields to not be fully random, in fact you want to set them on your own, you could
overwrite the values that are generated by `Fictitious` by providing the second argument:

```elixir
iex> Fictitious.fictionize(YourApp.Schema.Person, name: "some name", email: "some email")
{:ok, %YourApp.Schema.Person{
  __meta__: #Ecto.Schema.Metadata<:loaded, "persons">,
  id: 652,
  name: "some name",
  age: 1241,
  email: "some email",
  inserted_at: ~U[2020-05-31 20:11:21Z],
  updated_at: ~U[2020-05-31 20:11:42Z]
}}
```

### Schema with Changeset Validations

The previous schema in `Basics` section was a rather simple schema. What happen if we now decided to modify the `persons` schema
to add some field value validations in the `changeset/2` function as follow:

```elixir
defmodule YourApp.Schema.Person do
  use Ecto.Schema
  import Ecto.Changeset

  schema "persons" do
    field :name, :string
    field :age, :integer
    field :gender, :string
    field :email, :string

    timestamps()
  end

  @doc false
  def changeset(person, attrs) do
    person
    |> cast(attrs, [...])
    |> validate_inclusion(:gender, ["MALE", "FEMALE"]) # check if :gender is either "MALE" or "FEMALE"
    |> validate_email_format() # custom function to check if :email has the correct email format
  end
end
```

Having `validate_inclusion/3` and `validate_email_format/1` in the changeset would make the value of `:gender` and `:email` to be totally random hence
`Fictitious` will ignore any kind of validation in the changeset. Performing `fictionize/1` function to the new `persons` schema will still give you a
fictitious value for `:gender` and `:email`:

```elixir
iex> Fictitious.fictionize(YourApp.Schema.Person)
{:ok, %YourApp.Schema.Person{
  __meta__: #Ecto.Schema.Metadata<:loaded, "persons">,
  id: 1243,
  name: "2cBfxcqnB0B5iqhYvK83RamaDa8KM0PvPpT1kVao",
  age: 632,
  gender: "7U01hkeHYLLtSVNI3SPaSNSXrACVBsDRwFe13n6l7GzaAakcPkMtODZ2eiioqJHrWXITSLPMu7wJ8"
  email: "ixV5neQzcap5hq4dXycbt6Mj2fqgPLI3se6qXQbkmHOdoICyaX6",
  inserted_at: ~U[2020-04-31 06:19:27Z],
  updated_at: ~U[2020-04-31 06:19:27Z]
}}
```

In case you want the data to have the correct value for `:gender` and `:email` you need to specify them manually as previously has shown:

```elixir
iex> Fictitious.fictionize(YourApp.Schema.Person, gender: "MALE", email: "email@domain.com")
{:ok, %YourApp.Schema.Person{
  __meta__: #Ecto.Schema.Metadata<:loaded, "persons">,
  id: 564545,
  name: "xIzommg1lpwgRNBQCcGXLXdxORM7gXGqVIkC3gDL2As1DhxmhdejE0tXR2ImlrXN7j72nDO3Y",
  age: 235111,
  gender: "MALE"
  email: "email@domain.com",
  inserted_at: ~U[2020-04-31 06:19:27Z],
  updated_at: ~U[2020-04-31 06:19:27Z]
}}
```

### Associations

The true comfort of `Fictitious` comes when you encounter ecto schemas that have `%Ecto.Association.BelongsTo{}` relations to other schemas. Given
you have the following new `countries` schema as follow:

```elixir
defmodule YourApp.Schema.Country do
  use Ecto.Schema
  import Ecto.Changeset
  alias YourApp.Schema.Person

  schema "countries" do
    field :name, :string
    has_many :people, Person, foreign_key: :country_id

    timestamps()
  end

  ...
end
```

and in a `persons` schema we add `belongs_to` relation to `countries` as follow:

```elixir
defmodule YourApp.Schema.Person do
  use Ecto.Schema
  import Ecto.Changeset
  alias YourApp.Schema.Country

  schema "persons" do
    field :name, :string
    field :age, :integer
    field :gender, :string
    field :email, :string
    belongs_to :nationality, Country, references: :id, foreign_key: :country_id, type: :id

    timestamps()
  end
end
```

then depending on how you set the tables' relation in the DB, it is usually meant that for a person to exist it must belongs to a country hence
before any person record could be created, you must at least has one country record. It is possible that a person could exist without a country if
no constraint exist in the DB however this is the assumption that `Fictitous` will always make whenenver a schema has an `%Ecto.Association.BelongsTo{}`
relation. It will always assume that since `persons` belongs to `countries` then a country record must exist first.

calling `fictionize/1` to `YourApp.Schema.Country` will only makes a fictitious country:

```elixir
iex> Fictitious.fictionize(YourApp.Schema.Country)
{:ok, %YourApp.Schema.Country{
  __meta__: #Ecto.Schema.Metadata<:loaded, "countries">,
  id: 67,
  name: "B8LemwxB8ULP4NLUaFnKfwWkMmBYy8BTytkSN2PiL1UTO47yRM",
  people: #Ecto.Association.NotLoaded<association :people is not loaded>,
  inserted_at: ~U[2020-04-31 06:19:27Z],
  updated_at: ~U[2020-04-31 06:19:27Z]
}}
```

however calling `fictionize/1` to `YourApp.Schema.Person` will creates a person by creating the country first:

```elixir
iex> {:ok, person} = Fictitious.fictionize(YourApp.Schema.Person)
{:ok, %YourApp.Schema.Person{
  __meta__: #Ecto.Schema.Metadata<:loaded, "persons">,
  id: 725,
  name: "bElHKj9zVwnkLRpO4Y23yon9n80gm1yeAEL4PgtgkxBc0p2Y7C",
  age: 364,
  gender: "dF1O5Eq4ombjzah",
  email: "hpOXdOriGA9xaMhnwese40PqqL2Ine",
  nationality: #Ecto.Association.NotLoaded<association :nationality is not loaded>,
  inserted_at: ~U[2020-04-31 06:19:27Z],
  updated_at: ~U[2020-04-31 06:19:27Z]
}}

iex> YourApp.Repo.preload(person, :nationality)
%YourApp.Schema.Person{
  __meta__: #Ecto.Schema.Metadata<:loaded, "persons">,
  id: 725,
  name: "bElHKj9zVwnkLRpO4Y23yon9n80gm1yeAEL4PgtgkxBc0p2Y7C",
  age: 364,
  gender: "dF1O5Eq4ombjzah",
  email: "hpOXdOriGA9xaMhnwese40PqqL2Ine",
  nationality: %YourApp.Schema.Country{
    __meta__: #Ecto.Schema.Metadata<:loaded, "countries">,
    id: 401,
    name: "lcb1e86TY6RSccL6vPGjXOv43gnp1t",
    people: #Ecto.Association.NotLoaded<association :people is not loaded>
    inserted_at: ~U[2020-04-31 06:19:27Z],
    updated_at: ~U[2020-04-31 06:19:27Z]
  }
  inserted_at: ~U[2020-04-31 06:19:27Z],
  updated_at: ~U[2020-04-31 06:19:27Z]
}
```

Having the `belongs_to` associations to be created automatically removes the trouble of having to prepare other entities before
we could create the wanted the entity. This is usually happens a lot of time during preparing unit test data hence this is
one problem that `Fictitious` could solve and save us a lot of time. `Fictitious` ensures that you get the targeted or wanted entity to be created.

In case you want the created fictitious person to belongs to the previously created fictitious country then there are two ways you
could do that. First is by manually changing the `:country_id` as follows:

```elixir
iex> {:ok, country} = Fictitious.fictionize(YourApp.Schema.Country, name: "Indonesia")
{:ok, %YourApp.Schema.Country{
  __meta__: #Ecto.Schema.Metadata<:loaded, "countries">,
  id: 666409,
  name: "Indonesia",
  people: #Ecto.Association.NotLoaded<association :people is not loaded>,
  inserted_at: ~U[2020-04-31 06:19:27Z],
  updated_at: ~U[2020-04-31 06:19:27Z]
}}

iex> {:ok, person} = Fictitious.fictionize(YourApp.Schema.Person, country_id: country.id)
{:ok, %YourApp.Schema.Person{
  __meta__: #Ecto.Schema.Metadata<:loaded, "persons">,
  id: 5230,
  name: "FZcb5Q4zLOO4aMrdi1RblsEPpushgAn9zoPtfMbJWlsNe",
  age: 5768,
  gender: "FBTG2Ls4Fi9nD6oazpPjBqti5DfdmqyGTaQp5xlxjiH9B",
  email: "cgOACnmDFqbO5NxEZ0AUtwtjEfZBMcv3QzAq3esrcJHo7",
  nationality: #Ecto.Association.NotLoaded<association :nationality is not loaded>,
  inserted_at: ~U[2020-04-31 06:19:27Z],
  updated_at: ~U[2020-04-31 06:19:27Z]
}}
```
    
or second, by passing the whole `%YourApp.Schema.Country{}` struct as follows:

```elixir
iex> {:ok, country} = Fictitious.fictionize(YourApp.Schema.Country, name: "Indonesia")
{:ok, %YourApp.Schema.Country{
  __meta__: #Ecto.Schema.Metadata<:loaded, "countries">,
  id: 7914,
  name: "Indonesia",
  people: #Ecto.Association.NotLoaded<association :people is not loaded>,
  inserted_at: ~U[2020-04-31 06:19:27Z],
  updated_at: ~U[2020-04-31 06:19:27Z]
}}

iex> {:ok, person} = Fictitious.fictionize(YourApp.Schema.Person, nationality: country)
{:ok, %YourApp.Schema.Person{
  __meta__: #Ecto.Schema.Metadata<:loaded, "persons">,
  id: 451,
  name: "ZFvtidsGOPh6OymYJk529bL2QT9KMZic2A0ietddl2RWy",
  age: 150940,
  gender: "rHZYpbDgJQokDX2vSpSfWUmELrTb9f",
  email: "xmcuHrJvotjAQz6itQnZtoMp",
  nationality: #Ecto.Association.NotLoaded<association :nationality is not loaded>,
  inserted_at: ~U[2020-04-31 06:19:27Z],
  updated_at: ~U[2020-04-31 06:19:27Z]
}}
```

### Multiple Repos

Given you configured the `Fictitious` repo as follow:

```elixir
config :fictitious, :repo,
  default: YourApp.Repo,
  second_repo: YourApp.SecondRepo
``` 

then to use the second repo you could use `fictionize/2` or `fictionize/3` as follows:

```elixir
iex> Fictitious.fictionize(YourApp.Schema.Continent, :second_repo)
iex> Fictitious.fictionize(YourApp.Schema.Continent, :second_repo, name: "overwrite name")
```
