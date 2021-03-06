<!-- loio7d914317c0b64c23824bf932cc8a4ae1 -->

| loio |
| -----|
| 7d914317c0b64c23824bf932cc8a4ae1 |

<div id="loio">

view on: [demo kit nightly build](https://openui5nightly.hana.ondemand.com/#/topic/7d914317c0b64c23824bf932cc8a4ae1) | [demo kit latest release](https://openui5.hana.ondemand.com/#/topic/7d914317c0b64c23824bf932cc8a4ae1)</div>

## Extension for Data Aggregation

The OData V4 Model supports features of the OData Extension for Data Aggregation V4.0 specification.

The `$$aggregation` binding parameter at [`sap.ui.model.odata.v4.ODataModel#bindList`](https://openui5.hana.ondemand.com/#api/sap.ui.model.odata.v4.ODataModel/methods/bindList) holds the information needed for data aggregation. It may be changed by [`sap.ui.model.odata.v4.ODataListBinding#setAggregation`](https://openui5.hana.ondemand.com/#api/sap.ui.model.odata.v4.ODataListBinding/methods/setAggregation). It cannot be combined with an explicit system query option `$apply`, because it implicitly derives `$apply`. For more information, see the [OData Extension for Data Aggregation V4.0 specification](http://docs.oasis-open.org/odata/odata-data-aggregation-ext/v4.0/odata-data-aggregation-ext-v4.0.html).

For every aggregatable property, you can provide the name of the custom aggregate for a corresponding currency or unit of measure. That custom aggregate must return the single value of a unit in case there is only one, or `null` otherwise \("multi-unit situation"\). For SQL-based services, this might be implemented as follows:

 `CASE WHEN min(Unit) = max(Unit) THEN min(Unit) END` 

Normally, there is also a structural property of the same name as the custom aggregate, providing type information, etc.

The following client-side instance annotations can be used to access a node level or expansion state. For property bindings, a syntax like `{= %{@$ui5.node.level} }` is usually helpful, because automatic type determination is not available.

-   `@$ui5.node.level` – A non-negative integer which describes the node level; "0" is the single root node which corresponds to the grand total row, "1" are the top-level group nodes, etc.

-   `@$ui5.node.isExpanded` – A boolean which determines whether this node is currently expanded. `true` means yes, `false` means no, `undefined` means that \(the state is undefined because\) this node is a leaf. As an implementation detail, the annotation might simply be missing for leaves.

Two scenarios are supported:

-   You can provide properties for grouping and aggregation. An appropriate system query option `$apply` is derived from those. The list binding then still provides a flat list of contexts \("rows"\), but with additional aggregated properties \("columns"\). In addition, you can request grand total values for aggregatable properties. In this case, an extra row appears at the beginning of the flat list of contexts that contains the grand total values, as well as empty values for all other properties.

    ```
    
    					    **Example XML View With Grand Total**
    
    
    					    ``` js
        <table:Table fixedRowCount="1"
           rows="{
              path : '/BusinessPartners',
              parameters : {
                 $$aggregation : {
                    aggregate : {
                       SalesAmount : {
                          grandTotal : true,
                          unit : 'Currency'
                       }
                    },
                    group : {
                       Country : {}
                    }
                 },
                 $filter : 'SalesAmount gt 1000000',
                 $orderby : 'SalesAmount desc'
              }
           }">
           <table:Column template="Country">
              <Label text="Country"/>
           </table:Column>
           <table:Column hAlign="End" template="SalesAmount">
              <Label text="Sales Amount"/>
           </table:Column>
           <table:Column template="Currency">
              <Label text="Currency"/>
           </table:Column>
        </table:Table>
        ```
    
    
    				
    ```

-   You can provide group levels to determine a hierarchy of expandable group levels in addition to the leaf nodes determined by the groupable and aggregatable properties. To achieve this, specify the names of the group levels in the `groupLevels` property of `$$aggregation`.

    Group levels cannot be combined with the system query option `$count : true` and can only be combined with filtering before the aggregation \(see below\). Note how an `$orderby` option can address groups across all levels. For every aggregatable property, you can request subtotals and a grand total individually.

    ```
    
    					    **Example XML View With Hierarchy**
    
    
    					    ``` js
        <table:Table fixedRowCount="1"
           rows="{
              path : '/BusinessPartners',
              parameters : {
                 $$aggregation : {
                    aggregate : {
                       SalesAmount : {
                          grandTotal : true,
                          subtotals : true,
                          unit : 'Currency'
                       }
                    },
                    groupLevels : ['Country','Region','Segment']
                 },
                 $count : false,
                 $orderby : 'Country,Region desc,Segment',
                 filters : {path : \'Region\', operator : \'GE\', value1 : \'Mid\'}
              }
           }">
           <table:Column template="Country">
              <Label text="Country"/>
           </table:Column>
           <table:Column template="Region">
              <Label text="Region"/>
           </table:Column>
           <table:Column template="Segment">
              <Label text="Segment"/>
           </table:Column>
           <table:Column hAlign="End" template="SalesAmount">
              <Label text="Sales Amount"/>
           </table:Column>
           <table:Column template="Currency">
              <Label text="Currency"/>
           </table:Column>
        </table:Table>
        ```
    
    
    				
    ```


For aggregatable properties where grand total or subtotal values are requested, you can globally choose where these should be displayed:

-   at the bottom only,
-   at both the top and bottom,
-   at the top only \(default\).

Use the `grandTotalAtBottomOnly` or `subtotalsAtBottomOnly` property with values `true` or `false`, respectively, or simply omit it. For more information, see the [API Reference](https://openui5.hana.ondemand.com/#/api/sap.ui.model.odata.v4.ODataListBinding/methods/setAggregation) in the Demo Kit.

***

<a name="loio7d914317c0b64c23824bf932cc8a4ae1__section_igs_pyd_tkb"/>

### Filtering

Filters are provided to the list binding as described in [Filtering](Filtering_5338bd1.md). The `Filter` objects are analyzed automatically to perform the filtering before the aggregation where possible using the `filter()` transformation. The remaining filters, including the provided `$filter` parameter of the binding, are applied after the aggregation either via the system query option `$filter` or within the system query option `$apply`, using again the `filter()` transformation.

