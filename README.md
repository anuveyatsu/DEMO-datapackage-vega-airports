Flight connections of the airports in the US. This is an example DataPackage that demonstrates usage of "Vega Graph Specifications". This DataPackage is based on [U.S. Airports, 2008](https://mbostock.github.io/d3/talk/20111116/airports.html) by Mike Bostock.

## Views

To create graphs for your tabular DataPackage, the datapackage.json should include the views attribute that defines data views.

### Vega Graph Specifications

<script src="https://gist.github.com/anuveyatsu/f8860dc4bd3aa2f5697bd35a202aadfa.js"></script>
To use "Vega Graph Specification" `specType` inside `views` attribute should be set to `vega` - line 79. You can use almost the same specifications inside `spec` attribute, that are used for setting the vega graphs. Only difference is that in `data` property, all `url` and `path` attributes are moved out. Instead of that, `name` attribute is used to reference a dataset - lines 86, 89 and 110. Note, in lines 85-87 we created a new object inside `data` property - `{"name": "flights-airport"}` to reference it from other Vega data items (see lines 100 and 136).

Outside of `spec` attribute there are some other important parameters to note:

<table class="table table-bordered table-striped resource-summary">
  <thead>
   <tr>
     <th>Attribute</th>
     <th>Type</th>
     <th>Description</th>
   </tr>
  </thead>
  <tbody>
    <tr>
      <th>name</th>
      <td>String</td>
      <td>Unique identifier for view within list of views.</td>
    </tr>
    <tr>
      <th>title</th>
      <td>String</td>
      <td>Title for the graph.</td>
    </tr>
    <tr>
      <th>resources</th>
      <td>Array</td>
      <td>Data sources for this spec. It can be either resource name or index. By default it is the first resource.</td>
    </tr>
    <tr>
      <th>specType</th>
      <td>String</td>
      <td>Available options: simple, vega, plotly <strong>(Required)</strong>.</td>
    </tr>
  </tbody>
</table>
