---
permalink: /:categories/:year/:month/:day/:title/
---

# Custom step argument transformations with table arguments in SpecFlow by example

Consider to following SpecFlow scenario:

``` Gherkin

   Scenario: An identifier is equal to itself (Reflexive)
       Given the following identifiers
       | Id | Use | Label | System | Value |
       | 1 | Official | Description of identifier | urn://organization/domain/entity | '22'  |
       When I compare the identifier with id 1 with the identifier with id 1
       Then then identifiers are considered to be equal
```

The “Given” statement contains a table with an example of an Identifier instance and the step definition would be translated into this:

``` c#
    [Given(@"the following identifiers")]
    public void GivenTheFollowingIdentifiers(Table table)
    {

    }
```

What I would like to do is take the table and transform it into an IEnumerable<Identifier>, where the Identifier class looks like this:

``` c#
    public class Identifier
    {
        public int Id { get; set; }
        public string Use { get; set; }
        public string Label { get; set; }
        public Uri System { get; set; }
        public string Value { get; set; }
        public Period Period { get; set; }
        public Organization Organization { get; set; }
    }
```

In most cases, when the type only consists of primitive values, such as string or integers, one could use an assist helper to do this for you, like so:

``` c#
    [Given(@"the following identifiers")]
    public void GivenTheFollowingIdentifiers(Table table)
    {
        var identifiers = table.CreateSet<Identifier>();
    }
```

The assist helper will do a best effort attempt however, in this case for instance, each value for the System field (of type Uri) will be set to a null value, because SpecFlow doesn’t know how to transform the string into an Uri instance.
To handle this problem, SpecFlow has a concept called “Step argument transformations”. The documentation however doesn’t contain specifics on how to implement this in detail, so I am posting an example for the above situation here. Create a method that takes a Table and returns the type you need (an IEnumerable<Identifier> in my example) and decorate it with the StepArgumentTransformation attribute:

``` c#
    [StepArgumentTransformation]
    public IEnumerable<Identifier> TransformIdentifiers(Table identifiers)
    {
        return identifiers.Rows.Select(row => new Identifier
        {
            Id = int.Parse(row["Id"]),
            System = new Uri(row["System"]),
            Label = row["Label"],
            Use = row["Use"],
            Value = row["Value"]
        });
    }
```

As you can see it simply iterates the rows of the table, (of type TableRow) which are essentially dictionaries with the name of the column you use in the table in the scenario definition as the key. For each row an instance of an Identifier is created and added to the result.
