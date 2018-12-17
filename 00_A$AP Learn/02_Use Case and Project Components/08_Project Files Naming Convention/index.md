### Naming Folders and Files

Folder and file names are lower case, dash-separated (*kebab-case*). 

Designated sub-folders can be used to group files by the business component, e.g., `user` or `order`. 

File names include the name of the component the file represents. Plurals are used in folder names, but not in file names. 

To better identify files in stacked editor tabs, files names end with distinct suffixes:

- cud - (create-update-delete) data-modifying MongoDB handling methods 
- fetch - MongoDB queries
- model - MongoDB (`mongoose`) models and schemas
- type - GraphQL (`relay` Type) models
- mag - GraphQL mutate and get processors
- mutation - GraphQL *mutation* Type definitions
- query - GraphQL query Type definitions
- resolver - GraphQL *resolvers* - query handlers 
 

### Naming Fields, Variables and Functions

- MongoDB and GraphQL fields - lower case, underscore-delimited (*snake_case*). Field names for the same content in MongoDB and GraphQL are identical
- Variables and Functions - *camelCase*. Using different cases for fields vs. variables helps separating the two in the code
 

And now - on to the MongoDB stuff!
