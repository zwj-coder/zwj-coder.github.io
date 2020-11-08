# Database system summary

##  Different design for data model

### Hierarchical structure

Records' relationships form a treelike model, which is simple but inflexible because the relationship is confined to a one-to-many relaitonship.

The main drawback of hierarchical structure is :

1. Information is repeated.
2. Existence depends on parents.
3. Record-at-a-time access DML

### Network structure

Networks are more flexible than hierarchies but more complex.

Loading and recovering networks is more complex than hierarchies

### Relational

设计理念:

1. Store the data in a simple data structure (Table)
2. Access it through a high level set-at-a-time DML
3. No need for a physical storage proposal
