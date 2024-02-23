# Exploring Semspect

Using SemSpect for RDF (beta) on a dataset extracted from https://orcid.org

```
./semspect.sh orcid.nt
```

## Initial screen

At startup the browser displays information on classes and properties in the database.

### Classes
- http://schema.org/PropertyValue 1,277,592
- http://schema.org/CreativeWork 639.879
- [No class] 106,612
- http://schema.org/Organization 33,744
- http://schema.org/Person 27,901
- http://schema.org/PostalAddress 9,876

### Properties
- Any property 2,236,963
- http://schema.org/identifier 1,277,592
- http://schema.org/creator 770,533
- http://schema.org/sameAs 101,584
- http://schema.org/mainEntityOfPage 27,901
- http://schema.org/affiliation 20,691
- http://schema.org/alumniOf 17,427
- http://schema.org/address 9,876
- http://schema.org/url 7,779
- http://schema.org/funder 4,220

On the left hand side it shows the classes. Double clicking selects one of these.

## Resources

Clicking on selected class gives the option to “Show resource table of selected group”. This lists the values for literals for members of the class, and the URI for each member of the class (which may be a blank node).

For example, for http://schema.org/Person this view is created using this SPARQL query:

```
SELECT  (GROUP_CONCAT(DISTINCT str(?http_schema_org_alternateName_NotAggregated) ; separator=', ') AS ?http_schema_org_alternateName) ?resourceIRI (GROUP_CONCAT(DISTINCT str(?http_schema_org_name_NotAggregated) ; separator=', ') AS ?http_schema_org_name) (GROUP_CONCAT(DISTINCT str(?http_schema_org_familyName_NotAggregated) ; separator=', ') AS ?http_schema_org_familyName) (GROUP_CONCAT(DISTINCT str(?http_schema_org_givenName_NotAggregated) ; separator=', ') AS ?http_schema_org_givenName)
WHERE
  { { SELECT DISTINCT  ?group0
      WHERE
        { ?group0  a  <http://schema.org/Person> }
    }
    OPTIONAL
      { ?group0  <http://schema.org/alternateName>  ?http_schema_org_alternateName_Optional }
    BIND(coalesce(?http_schema_org_alternateName_Optional, "") AS ?http_schema_org_alternateName_NotAggregated)
    OPTIONAL
      { ?group0  <http://schema.org/name>  ?http_schema_org_name_Optional }
    BIND(coalesce(?http_schema_org_name_Optional, "") AS ?http_schema_org_name_NotAggregated)
    OPTIONAL
      { ?group0  <http://schema.org/familyName>  ?http_schema_org_familyName_Optional }
    BIND(coalesce(?http_schema_org_familyName_Optional, "") AS ?http_schema_org_familyName_NotAggregated)
    OPTIONAL
      { ?group0  <http://schema.org/givenName>  ?http_schema_org_givenName_Optional }
    BIND(coalesce(?http_schema_org_givenName_Optional, "") AS ?http_schema_org_givenName_NotAggregated)
    BIND(?group0 AS ?resourceIRI)
  }
GROUP BY ?resourceIRI
OFFSET  0
LIMIT   50
```

## Connections

Clicking on the triangle to the right lists class connected to the current class. You can click on a particular connection (e.g., schema:creator to schema:CreativeWork), which then displays that connection. 

The number of connected classes is given by a SPARQL query that times out in Oxigraph, e.g.:

```
SELECT DISTINCT  ?group2
WHERE
  { ?group2  a  <http://schema.org/Organization>
    { SELECT DISTINCT  ?group0
      WHERE
        { ?group0  a  <http://schema.org/Person> }
    }
    ?group0  <http://schema.org/affiliation>  ?group2
  }

```

You can get a list of these connections with this SPARQL query (which times out in Oxigraph):

```
SELECT  ?rightResourceIRI ?linkingProperty ?leftResourceIRI (GROUP_CONCAT(DISTINCT str(?rightResourceCaption_NotAggregated) ; separator=', ') AS ?rightResourceCaption) (GROUP_CONCAT(DISTINCT str(?leftResourceCaption_NotAggregated) ; separator=', ') AS ?leftResourceCaption)
WHERE
  { { { SELECT DISTINCT  ?group1
        WHERE
          { ?group1  a  <http://schema.org/CreativeWork>
            { SELECT DISTINCT  ?group0
              WHERE
                { ?group0  a  <http://schema.org/Person> }
            }
            ?group1  <http://schema.org/creator>  ?group0
          }
      }
      { SELECT DISTINCT  ?group0
        WHERE
          { ?group0  a  <http://schema.org/Person> }
      }
      ?group1  <http://schema.org/creator>  ?group0
      OPTIONAL
        {   { ?path_group0_group1
                        <http://www.w3.org/2002/07/owl#annotatedSource>  ?group1 ;
                        <http://www.w3.org/2002/07/owl#annotatedProperty>  <http://schema.org/creator> ;
                        <http://www.w3.org/2002/07/owl#annotatedTarget>  ?group0
            }
          UNION
            { ?path_group0_group1
                        <http://www.w3.org/1999/02/22-rdf-syntax-ns#subject>  ?group1 ;
                        <http://www.w3.org/1999/02/22-rdf-syntax-ns#predicate>  <http://schema.org/creator> ;
                        <http://www.w3.org/1999/02/22-rdf-syntax-ns#object>  ?group0
            }
        }
      BIND(?group1 AS ?rightResourceIRI)
      BIND(?group0 AS ?leftResourceIRI)
      BIND(<http://schema.org/creator> AS ?linkingProperty)
      OPTIONAL
        { ?rightResourceIRI
                    <http://schema.org/alternateName>  ?rightResourceCaption_Optional
        }
      BIND(coalesce(?rightResourceCaption_Optional, "") AS ?rightResourceCaption_NotAggregated)
      OPTIONAL
        { ?leftResourceIRI
                    <http://schema.org/alternateName>  ?leftResourceCaption_Optional
        }
      BIND(coalesce(?leftResourceCaption_Optional, "") AS ?leftResourceCaption_NotAggregated)
    }
  }
GROUP BY ?leftResourceIRI ?linkingProperty ?rightResourceIRI
OFFSET  0
LIMIT   50

```


