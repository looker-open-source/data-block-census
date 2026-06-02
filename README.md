# Census Data Block

This is not an officially supported Google product. This project is not eligible for the [Google Open Source Software Vulnerability Rewards Program](https://bughunters.google.com/open-source-security).

The U.S. Census Bureau’s [American Community Survey (ACS)](https://www.census.gov/programs-surveys/acs) is an annual nationwide survey that collects and produces information on social, economic, housing, and demographic characteristics in the U.S. This Data Block is based on a [publicly available dataset hosted in Google BigQuery](https://console.cloud.google.com/marketplace/details/united-states-census-bureau/acs?id=1282ab4c-78a4-4da5-8af8-cd693fe390ab) as part of the [Google Cloud Public Datasets Program](https://cloud.google.com/public-datasets?_ga=2.233975447.-840160752.1587661252). Here we reference the most recent [5 year estimates](https://www.census.gov/programs-surveys/acs/guidance/estimates.html) for each geographic region currently available in BigQuery.

You'll find two types of view files in this Data Block: those related to geography (in the `geography` folder) and those that group fields into categories like `employment` and `population` (in the views folder). The Census Bureau designates  `GEOID` identifiers for each geographic region. `GEOID`s are hierarchical, allowing the `geographic` views to be joined via their respective `geo_id` fields to more granular levels.

This block includes the following premade explores. Each explore includes accompanying map layers based on Census Bureau's `GEOID` designations:

- ACS Census Data
  - State
  - County
  - Census Tract
  - Block Group
- Congressional Districts
- Core-Based Statistical Areas (CBSA)
- Places
- Public Use Microdata Areas (PUMA)
- Elementary, Secondary, and Unified School Districts (separate explores)
- Zip Code Tabulation Areas (ZCTA)

**Please refer to [this link](https://www.census.gov/programs-surveys/geography/guidance/geo-identifiers.html) for more detail regarding `GEOID` designations.** Additionally, you can scroll to the bottom of this document for a glossary of geographic terms.

***

### Importing this block into other projects
Views and explores from this Data Block can be brought into your other projects by using [project import](https://docs.looker.com/data-modeling/learning-lookml/importing-projects) via [extends](https://docs.looker.com/data-modeling/learning-lookml/extends) or [refinements](https://docs.looker.com/data-modeling/learning-lookml/refinements) syntax. Please refer to the examples below for more detail.

#### Project Import Examples
Imagine we have a project with an `orders` explore which we join to an `address` view. We're interested in learning more about the demographics of a few areas where we have a high number of orders and decide to incorporate this Data Block into our original project.

The original explore looks like this:

```
explore: orders {
  join: address {
    sql_on: ${orders.address_id} = ${address.id}
    relationship: many_to_one
  }
}
```

We’ll first want to include all the geographic layers from this Data Block by adding this `include` header to our model file after setting up the [project’s manifest for project import](https://docs.looker.com/data-modeling/learning-lookml/importing-projects#create_a_project_manifest_file):

```
include: "//marketplace_block_acs_census_bigquery/geography/*.lkml"
```
We now have a few options for integrating this Data Block with our own project:

#### **1. Importing Premade Explores**
  We can easily incorporate content from this Data Block by extending a premade explore, provided we have the proper fields to join on. Below shows how this would look if we included Zip Code Tabulation Area (ZCTA) data in our original `orders` explore:

```
include: "//marketplace_block_acs_census_bigquery/geography/*.lkml"

explore: orders {
  extends: [zcta]
  join: address {
    sql_on: ${orders.address_id} = ${address.id}
    relationship: many_to_one
  }
  join: zcta {
   sql_on: ${zcta.zcta} = ${address.zip_code}
   relationship: one_to_many
  }
}
```
Here we extend the premade `zcta` explore into `orders` and join them using `zcta.zcta` and `address.zip_code` fields.

#### **2. Importing Individual Geographic Layers**
  We can also import geography views individually. In most cases, importing individual geographies requires no additional view declarations --  simply import the view and join it to an explore much like the above example. However, If using `state`, `county` or `census_tract` geographies we’ll need to create an additional view to populate the imported layer with fields. Let’s take a look on how we’d import just the `state` view into our project:

```
include: "//marketplace_block_acs_census_bigquery/geography/*.lkml"

# We’ll need an additional include statement to bring in the view files
include: “//marketplace_block_acs_census_bigquery/views/*.lkml”

#Here’s the additional view to populate the state geography layer with fields
view: state_extended {
 extends: [state, gender, housing, race, education,  employment, population, family ]
}

explore: orders {
  join: address {
    sql_on: ${orders.address_id} = ${address.id}
    relationship: many_to_one
  }
  join: state_extended {
   sql_on: ${state_extended.state_abbreviation} = ${address.state}
   relationship: one_to_many
 }
}

```

Again, the need for the additional view is only required for importing the `state`, `county` or `census_tract` geographies alone. For importing other views, we’ll simply join the view to our explore. For example, were we to include just the `places` view:

```
include: "//marketplace_block_acs_census_bigquery/geography/*.lkml"

explore: orders {
  join: address {
    sql_on: ${orders.address_id} = ${address.id}
    relationship: many_to_one
  }
  join: places {
    sql_on: ${places.place_name} = ${address.census_place}
    relationship: one_to_many
  }
```

***

### Geography Glossary
The following definitions are sourced from [Appendix A of Census 2010](https://www.census.gov/prod/cen2010/doc/sf1.pdf#page=605)

- **Census Tract**
  - Census tracts are small, relatively permanent statistical subdivisions of a county or statistically equivalent entity delineated by local participants as part of the U.S. Census Bureau’s Participant Statistical Areas Program. The U.S. Census Bureau delineated census tracts where no local participant existed or where a local or tribal government declined to participate. The primary purpose of census tracts is to provide a stable set of geographic units for the presentation of decennial census data.

- **Block Group**
  - A block group (BG) is a cluster of census blocks having the same first digit of their four-digit identifying numbers within a census tract. For example, block group 3 (BG 3) within a census tract includes all blocks numbered from 3000 to 3999. BGs generally contain between 600 and 3,000 people, with an optimum size of 1,500 people.

- **Public Use Microdata Area**
  - A public use microdata area (PUMA) is a decennial census area for which the U.S. Census Bureau provides specially selected extracts of raw data from a small sample of long-form census records that are screened to protect confidentiality. These extracts are referred to as ‘‘public use microdata sample (PUMS)’’ files. Since 1960, data users have been using these files to create their own statistical tabulations and data summaries.

- **Place**
  - Places, for the reporting of decennial census data, include census designated places, consolidated cities, and incorporated places. Each place is assigned a five-digit Federal Information Processing Standards (FIPS) code, based on the alphabetical order of the place name within each state.

- **School Districts**
  - School districts are geographic entities within which state, county, or local officials or the Department of Defense provide public educational services for the areas residents. The U.S. Census Bureau obtains the boundaries and names for school districts from state officials. Each school district is assigned a five-digit code that is unique within state. School district codes are assigned by the Department of Education and are not necessarily in alphabetical order by school district name.

- **Zip Code Tabulation Areas (ZCTA)**
  - A ZIP Code® tabulation area (ZCTA™) is a statistical geographic entity that approximates the delivery area for a U.S. Postal Service five-digit or three-digit ZIP Code. ZCTAs are aggregations of census blocks that have the same predominant ZIP Code associated with the residential mailing addresses in the U.S. Census Bureau’s Master Address File.

- **Core-Based Statistical Area (CBSA)**
  - Core Based Statistical Areas (CBSAs) consist of the county or counties or equivalent entities associated with at least one core (urbanized area or urban cluster) of at least 10,000 population, plus adjacent counties having a high degree of social and economic integration with the core as measured through commuting ties with the counties associated with the core. The general concept of a CBSA is that of a core area containing a substantial population nucleus, together with adjacent communities having a high degree of economic and social integration with that core.
