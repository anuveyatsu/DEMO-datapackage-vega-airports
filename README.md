This is an example dataset that demonstrates how to build visualizations using the "Vega Graph Spec". We are using [Flight connections of the airports in the US][airports] - one of the examples from [vega editor][editor] - and displaying it here, on DataHub with small modifications in vega-spec.

## Views

We assume that you are familiar with what [datapackage.json][datapackage.json] is and its specifications.

To create graphs for your tabular data, the `datapackage.json` should include the `views` attribute that is used for visualizations.

If you are familiar with [Vega][vega] specifications, you probably would like to use all its features. To use it, inside `views` you should set `specType` to `"vega"` and define some graph specifications in `spec` property. See below for more details.

### Vega Graph Specifications

This is the `views` property that is used for this page:

```
{
  ...,
  "views": [
    {
      "name": "airports-map-interactive",
      "title": "Airports connections in the US",
      "resources": ["states", 1, 2],
      "specType": "vega",
      "spec": {
        "$schema": "https://vega.github.io/schema/vega/v3.0.json",
        "width": 900,
        "height": 560,
        "padding": {
          "top": 25,
          "left": 0,
          "right": 0,
          "bottom": 0
        },
        "autosize": "none",
        "signals": [
          {
            "name": "scale",
            "value": 1200,
            "bind": {
              "input": "range",
              "min": 500,
              "max": 3000
            }
          },
          {
            "name": "translateX",
            "value": 450,
            "bind": {
              "input": "range",
              "min": -500,
              "max": 1200
            }
          },
          {
            "name": "translateY",
            "value": 260,
            "bind": {
              "input": "range",
              "min": -300,
              "max": 700
            }
          },
          {
            "name": "shape",
            "value": "line",
            "bind": {
              "input": "radio",
              "options": [
                "line",
                "curve"
              ]
            }
          },
          {
            "name": "hover",
            "value": null,
            "on": [
              {
                "events": "@cell:mouseover",
                "update": "datum"
              },
              {
                "events": "@cell:mouseout",
                "update": "null"
              }
            ]
          },
          {
            "name": "title",
            "value": "U.S. Airports, 2008",
            "update": "hover ? hover.name + ' (' + hover.iata + ')' : 'U.S. Airports, 2008'"
          },
          {
            "name": "cell_stroke",
            "value": null,
            "on": [
              {
                "events": "dblclick",
                "update": "cell_stroke ? null : 'brown'"
              },
              {
                "events": "mousedown!",
                "update": "cell_stroke"
              }
            ]
          }
        ],
        "data": [
          {"name": "flights-airport"},
          {
            "name": "states",
            "format": {
              "type": "topojson",
              "feature": "states"
            },
            "transform": [
              {
                "type": "geopath",
                "projection": "projection"
              }
            ]
          },
          {
            "name": "traffic",
            "source": "flights-airport",
            "format": {
              "type": "csv",
              "parse": "auto"
            },
            "transform": [
              {
                "type": "aggregate",
                "groupby": [
                  "origin"
                ],
                "fields": [
                  "count"
                ],
                "ops": [
                  "sum"
                ],
                "as": [
                  "flights"
                ]
              }
            ]
          },
          {
            "name": "airports",
            "format": {
              "type": "csv",
              "parse": "auto"
            },
            "transform": [
              {
                "type": "lookup",
                "from": "traffic",
                "key": "origin",
                "fields": [
                  "iata"
                ],
                "as": [
                  "traffic"
                ]
              },
              {
                "type": "filter",
                "expr": "datum.traffic != null"
              },
              {
                "type": "geopoint",
                "projection": "projection",
                "fields": [
                  "longitude",
                  "latitude"
                ]
              },
              {
                "type": "filter",
                "expr": "datum.x != null && datum.y != null"
              },
              {
                "type": "voronoi",
                "x": "x",
                "y": "y"
              },
              {
                "type": "collect",
                "sort": {
                  "field": "traffic.flights",
                  "order": "descending"
                }
              }
            ]
          },
          {
            "name": "routes",
            "source": "flights-airport",
            "format": {
              "type": "csv",
              "parse": "auto"
            },
            "transform": [
              {
                "type": "filter",
                "expr": "hover && hover.iata == datum.origin"
              },
              {
                "type": "lookup",
                "from": "airports",
                "key": "iata",
                "fields": [
                  "origin",
                  "destination"
                ],
                "as": [
                  "source",
                  "target"
                ]
              },
              {
                "type": "filter",
                "expr": "datum.source && datum.target"
              },
              {
                "type": "linkpath",
                "shape": {
                  "signal": "shape"
                }
              }
            ]
          }
        ],
        "projections": [
          {
            "name": "projection",
            "type": "albersUsa",
            "scale": {
              "signal": "scale"
            },
            "translate": [
              {
                "signal": "translateX"
              },
              {
                "signal": "translateY"
              }
            ]
          }
        ],
        "scales": [
          {
            "name": "size",
            "type": "linear",
            "domain": {
              "data": "traffic",
              "field": "flights"
            },
            "range": [
              16,
              1000
            ]
          }
        ],
        "marks": [
          {
            "type": "path",
            "from": {
              "data": "states"
            },
            "encode": {
              "enter": {
                "fill": {
                  "value": "#dedede"
                },
                "stroke": {
                  "value": "white"
                }
              },
              "update": {
                "path": {
                  "field": "path"
                }
              }
            }
          },
          {
            "type": "symbol",
            "from": {
              "data": "airports"
            },
            "encode": {
              "enter": {
                "size": {
                  "scale": "size",
                  "field": "traffic.flights"
                },
                "fill": {
                  "value": "steelblue"
                },
                "fillOpacity": {
                  "value": 0.8
                },
                "stroke": {
                  "value": "white"
                },
                "strokeWidth": {
                  "value": 1.5
                }
              },
              "update": {
                "x": {
                  "field": "x"
                },
                "y": {
                  "field": "y"
                }
              }
            }
          },
          {
            "type": "path",
            "name": "cell",
            "from": {
              "data": "airports"
            },
            "encode": {
              "enter": {
                "fill": {
                  "value": "transparent"
                },
                "strokeWidth": {
                  "value": 0.35
                }
              },
              "update": {
                "path": {
                  "field": "path"
                },
                "stroke": {
                  "signal": "cell_stroke"
                }
              }
            }
          },
          {
            "type": "path",
            "interactive": false,
            "from": {
              "data": "routes"
            },
            "encode": {
              "enter": {
                "path": {
                  "field": "path"
                },
                "stroke": {
                  "value": "black"
                },
                "strokeOpacity": {
                  "value": 0.35
                }
              }
            }
          },
          {
            "type": "text",
            "interactive": false,
            "encode": {
              "enter": {
                "x": {
                  "value": 895
                },
                "y": {
                  "value": 0
                },
                "fill": {
                  "value": "black"
                },
                "fontSize": {
                  "value": 20
                },
                "align": {
                  "value": "right"
                }
              },
              "update": {
                "text": {
                  "signal": "title"
                }
              }
            }
          }
        ]
      }
    }
  ]
}
```

<br>

You can use almost the same specifications inside `spec` attribute, that are used for setting the Vega graphs. Only difference is that in `data` property, `url` and `path` attributes are moved out. Instead we use `name` attribute to reference to a resource. Note, how we created a new object inside `data` property - `{"name": "flights-airport"}` to reference it to a resource (this names are identical to names of resources):

```
  ...
  "spec": {
    ...
    "data": [
      {
        "name": "flights-airport"
      }
      ...
    ]
  }
```

Outside of `spec` attribute there are some other important attributes to note:

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

[vega]: https://vega.github.io/vega/
[airports]: https://mbostock.github.io/d3/talk/20111116/airports.html
[editor]: https://vega.github.io/editor/#/examples/vega/airport-connections
[datapackage.json]: http://specs.frictionlessdata.io/data-package/
