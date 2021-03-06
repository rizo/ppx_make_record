# ppx_obj_make

> Unreleased

This syntax extension leverages ReasonML's object literal notation to simplify
initialization of labeled multi-parameter functions. In particular it is useful
for large nested configuration DSLs where the regular [labeled function application
syntax](https://reasonml.github.io/docs/en/function#optional-labeled-arguments)
might be cumbersome to use because of the dangling `()` required for optional
argument erasure.


## Quickstart

To install `ppx_obj_make` in an [esy](https://esy.sh) project, add the following
dependency to your `package.json` file:

```
  "dependencies": {
    "ppx_obj_make": "github:rizo/ppx_obj_make#master"
  }
```

In your [dune](https://dune.build/) project add the following preprocess
configuraton to your `dune` file:

```
(preprocess (pps ppx_obj_make))
```

## Usage

Any module containing a function named `make` with labeled arguments can be used
with this extension.

```reason
open Printf;

module Person = {
  let make = (~name, ~age, ()) =>
    printf("Hi %s! You are %d years old.\n", name, age);
};

/* Use object literal notation to call `Person.make`. */
Person { "name": "Dylan", "age": 32 };
```


## Examples

Consider this example in ReasonML for a Kubernetes deployment (that uses the
[rekube](https://github.com/rizo/rekube) library). The translation
is only applied to JSON-literals.

```reason
let app = (~namespace, ~version) =>
  Deployment {
    "api_version": "extensions/v1beta1",
    "kind": "Deployment",
    "metadata": Meta { name, namespace },
    "spec": Deployment_spec {
      "replicas": 1,
      "template": Pod_template_spec {
        "metadata": Meta {
          "name": name,
          "labels": [("app", name)],
        },
        "spec": Pod_spec {
          "containers": [
            "Container" {
              "name",
              "image": "my-company/my-app:" ++ version,
              "command": ["app"],
              "args": [
                "--verbosity=debug",
                "--listen-prometheus=9090"
              ],
              "ports": [Port { "name": "metrics", "container_port": 9090 }],
              "resources": Resources {
                "limits":   [cpu("100m"), memory("500Mi")],
                "requests": [cpu("500m"), memory("1Gi")]
              }
            }
          ]
        }
      }
    }
  };
```

This is translated by the preprocessor into:

```reason
let app = (~namespace, ~version) =>
  Deployment.make(
    ~api_version="extensions/v1beta1",
    ~kind="Deployment",
    ~metadata=Meta.make(~name, ~namespace, ()),
    ~spec=Deployment_spec.make(
      replicas: 1,
      template: Pod_template_spec.make(
        ~metadata=Meta.make(
          ~name,
          ~labels=[("app", name)],
          ()
        ),
        ~spec=Pod_spec.make(
          ~containers: [
            Container.make(
              ~name,
              ~image="my-company/my-app:" ++ version,
              ~command=["app"],
              ~args=[
                "--verbosity=debug",
                "--listen-prometheus=9090"
              ],
              ~ports-[Port.make(~name="metrics", ~container_port=9090, ())],
              ~resources=Resources.make(
                limits:   [cpu("100m"), memory("500Mi")],
                requests: [cpu("500m"), memory("1Gi")],
                ()
              ),
              ()
            )
          ],
          ()
        ),
        ()
      ),
      ()
    ),
    ()
  );
```
