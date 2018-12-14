### Naming Folders and Files

All names are lower case, dash-separated (*kebab-case*). 

Separate sub-folders can be used to group files by the business component. The name of the component is present in the file name. Plurals are used in folder names, but not in file names. 

To better identify files in the VSC editor tabs, files names end with distinct suffixes:
- cud - (create-update-delete) data-modifying MongoDB handling methods 
- fetch - MongoDB queries
- model - MongoDB (`mongoose`) models (also, files that contain Schemas only)
- type - GraphQL (`relay` Type) models
- mag - GraphQL mutate and get processors
- mutation - GraphQL mutation Type definitions
- query - GraphQL query Type definitions
- resolver - GraphQL *resolvers* - query handlers 
 

### Naming Fields, Variables and Functions

- MongoDB and GraphQL fields - lower case, underscore-delimited (*snake_case*). Generally, keeping the matching names in both MongoDB and in the GraphQL data-serving layer simplifies the app code as GraphQL recognizes the names by default
- Variables and Functions - *camelCase*. Using different cases for fields vs. variables helps separating the two in the code