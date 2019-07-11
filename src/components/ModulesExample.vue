<template>
  <div class="modules-example">
    <div al-app id="modules">
      <div
        class="module-list"
        al-repeat="(key, val) in modules"
        key="key"
        al-on.click="openModule(key)"
      >
        {{ "\{\{ key \}\}" }}
      </div>
      <button al-on.click="addModule()">Add</button>
    </div>
    <div id="rete" class="node-editor"></div>
  </div>
</template>

<script>
import Rete from "rete";
import alight from "alight";
import ConnectionPlugin from "rete-connection-plugin";
import AlightRenderPlugin from "rete-alight-render-plugin";
import ContextMenuPlugin from "rete-context-menu-plugin";
import ModulePlugin from "rete-module-plugin";
import AreaPlugin from "rete-area-plugin";

var numSocket = new Rete.Socket("Number");
var floatSocket = new Rete.Socket("Float");

class TextControl extends Rete.Control {
  constructor(emitter, key, readonly, type = "text") {
    super(key);
    this.emitter = emitter;
    this.key = key;
    this.type = type;
    this.template = `<input type="${type}" :readonly="readonly" :value="value" @input="change($event)"/>`;

    this.scope = {
      value: null,
      readonly,
      change: this.change.bind(this)
    };
  }

  onChange() {}

  change(e) {
    this.scope.value =
      this.type === "number" ? +e.target.value : e.target.value;
    this.update();
    this.onChange();
  }

  update() {
    if (this.key) this.putData(this.key, this.scope.value);
    this.emitter.trigger("process");
    this._alight.scan();
  }

  mounted() {
    this.scope.value =
      this.getData(this.key) || (this.type === "number" ? 0 : "...");
    this.update();
  }

  setValue(val) {
    this.scope.value = val;
    this._alight.scan();
  }
}

class InputComponent extends Rete.Component {
  constructor() {
    super("Input");
    this.module = {
      nodeType: "input",
      socket: numSocket
    };
  }

  builder(node) {
    var out1 = new Rete.Output("output", "Number", numSocket);
    var ctrl = new TextControl(this.editor, "name");
    return node.addControl(ctrl).addOutput(out1);
  }
}

class ModuleComponent extends Rete.Component {
  constructor() {
    super("Module");
    this.module = {
      nodeType: "module"
    };
  }

  builder(node) {
    var ctrl = new TextControl(this.editor, "module");
    ctrl.onChange = () => {
      console.log(this);
      this.updateModuleSockets(node);
      node._alight.scan();
    };
    return node.addControl(ctrl);
  }

  change(node, item) {
    node.data.module = item;
    this.editor.trigger("process");
  }
}

class OutputComponent extends Rete.Component {
  constructor() {
    super("Output");
    this.module = {
      nodeType: "output",
      socket: numSocket
    };
  }

  builder(node) {
    var inp = new Rete.Input("input", "Number", numSocket);
    var ctrl = new TextControl(this.editor, "name");

    return node.addControl(ctrl).addInput(inp);
  }
}

class OutputFloatComponent extends Rete.Component {
  constructor() {
    super("Float Output");
    this.module = {
      nodeType: "output",
      socket: floatSocket
    };
  }

  builder(node) {
    var inp = new Rete.Input("float", "Float", floatSocket);
    var ctrl = new TextControl(this.editor, "name");

    return node.addControl(ctrl).addInput(inp);
  }
}

class NumComponent extends Rete.Component {
  constructor() {
    super("Number");
  }

  builder(node) {
    var out1 = new Rete.Output("num", "Number", numSocket);
    var ctrl = new TextControl(this.editor, "num", false, "number");

    return node.addControl(ctrl).addOutput(out1);
  }

  worker(node, inputs, outputs) {
    outputs["num"] = node.data.num;
  }
}

class AddComponent extends Rete.Component {
  constructor() {
    super("Add");
  }

  builder(node) {
    var inp1 = new Rete.Input("num1", "Number", numSocket);
    var inp2 = new Rete.Input("num2", "Number", numSocket);
    var out = new Rete.Output("num", "Number", numSocket);

    inp1.addControl(new TextControl(this.editor, "num1", false, "number"));
    inp2.addControl(new TextControl(this.editor, "num2", false, "number"));

    return node
      .addInput(inp1)
      .addInput(inp2)
      .addControl(new TextControl(this.editor, "preview", true))
      .addOutput(out);
  }

  worker(node, inputs, outputs, { silent } = {}) {
    var n1 = inputs["num1"].length ? inputs["num1"][0] : node.data.num1;
    var n2 = inputs["num2"].length ? inputs["num2"][0] : node.data.num2;
    var sum = n1 + n2;

    if (!silent)
      this.editor.nodes
        .find(n => n.id == node.id)
        .controls.get("preview")
        .setValue(sum);

    outputs["num"] = sum;
  }

  created(node) {
    console.log("created", node);
  }

  destroyed(node) {
    console.log("destroyed", node);
  }
}

export default {
  data() {
    return {
      modules: {
        "index.rete": {
          data: {
            id: "demo@0.1.0",
            nodes: {
              "4": {
                id: 4,
                data: {
                  module: "module1.rete"
                },
                inputs: {
                  "2": {
                    connections: [
                      {
                        node: 12,
                        output: "num",
                        data: {}
                      }
                    ]
                  },
                  "...": {
                    connections: [
                      {
                        node: 12,
                        output: "num",
                        data: {}
                      }
                    ]
                  }
                },
                outputs: {
                  "...": {
                    connections: [
                      {
                        node: 5,
                        input: "num1",
                        data: {}
                      },
                      {
                        node: 5,
                        input: "num2",
                        data: {}
                      }
                    ]
                  }
                },
                position: [854, 159],
                name: "Module"
              },
              "5": {
                id: 5,
                data: {
                  num1: 0,
                  num2: 0
                },
                inputs: {
                  num1: {
                    connections: [
                      {
                        node: 4,
                        output: "...",
                        data: {}
                      }
                    ]
                  },
                  num2: {
                    connections: [
                      {
                        node: 4,
                        output: "...",
                        data: {}
                      }
                    ]
                  }
                },
                outputs: {
                  num: {
                    connections: []
                  }
                },
                position: [1217, 123],
                name: "Add"
              },
              "12": {
                id: 12,
                data: {
                  num: 1
                },
                inputs: {},
                outputs: {
                  num: {
                    connections: [
                      {
                        node: 4,
                        input: "...",
                        data: {}
                      },
                      {
                        node: 4,
                        input: "2",
                        data: {}
                      }
                    ]
                  }
                },
                position: [410, 212],
                name: "Number"
              }
            }
          }
        },
        "module1.rete": {
          data: {
            id: "demo@0.1.0",
            nodes: {
              "1": {
                id: 1,
                data: {
                  name: "..."
                },
                inputs: {},
                outputs: {
                  output: {
                    connections: [
                      {
                        node: 9,
                        input: "num1",
                        data: {}
                      }
                    ]
                  }
                },
                position: [348, 78],
                name: "Input"
              },
              "2": {
                id: 2,
                data: {
                  name: "..."
                },
                inputs: {
                  input: {
                    connections: [
                      {
                        node: 9,
                        output: "num",
                        data: {}
                      }
                    ]
                  }
                },
                outputs: {},
                position: [1086, 202],
                name: "Output"
              },
              "8": {
                id: 8,
                data: {
                  name: "2"
                },
                inputs: {},
                outputs: {
                  output: {
                    connections: [
                      {
                        node: 9,
                        input: "num2",
                        data: {}
                      }
                    ]
                  }
                },
                position: [285, 320],
                name: "Input"
              },
              "9": {
                id: 9,
                data: {
                  num1: 0,
                  num2: 0
                },
                inputs: {
                  num1: {
                    connections: [
                      {
                        node: 1,
                        output: "output",
                        data: {}
                      }
                    ]
                  },
                  num2: {
                    connections: [
                      {
                        node: 8,
                        output: "output",
                        data: {}
                      }
                    ]
                  }
                },
                outputs: {
                  num: {
                    connections: [
                      {
                        node: 2,
                        input: "input",
                        data: {}
                      }
                    ]
                  }
                },
                position: [730, 275],
                name: "Add"
              }
            }
          }
        }
      },
      currentModule: {},
      editor: null
    };
  },
  methods: {
    async openModule(name) {
      this.currentModule.data = this.editor.toJSON();
      this.currentModule = this.modules[name];
      await this.editor.fromJSON(this.currentModule.data);
      this.editor.trigger("process");
    },
    addModule() {
      this.modules["module" + Object.keys(this.modules).length + ".rete"] = {
        data: this.initialData()
      };
    },
    initialData() {
      return { id: "demo@0.1.0", nodes: {} };
    }
  },
  mounted() {
    var container = document.querySelector("#rete");

    alight("#modules", {
      modules: this.modules,
      addModule: this.addModule,
      openModule: this.openModule
    });

    this.editor = new Rete.NodeEditor("demo@0.1.0", container);
    this.editor.use(ConnectionPlugin, { curvature: 0.4 });
    this.editor.use(AlightRenderPlugin);
    this.editor.use(ContextMenuPlugin);

    var engine = new Rete.Engine("demo@0.1.0");

    this.editor.use(ModulePlugin, { engine, modules: this.modules });
    //engine.use(ProfilerPlugin, { this.editor, enabled: true });
    [
      new NumComponent(),
      new AddComponent(),
      new InputComponent(),
      new ModuleComponent(),
      new OutputComponent(),
      new OutputFloatComponent()
    ].map(c => {
      this.editor.register(c);
      engine.register(c);
    });
    this.editor.on("process connectioncreated connectionremoved", async () => {
      if (this.editor.silent) return;

      await engine.abort();
      await engine.process(this.editor.toJSON());
    });
    this.editor.view.resize();
    this.openModule("index.rete").then(() => {
      AreaPlugin.zoomAt(this.editor);
    });
  }
};
</script>

<style lang="scss" scoped>
// Since node editor's parent must have a set height
.modules-example {
  height: 100vh;
  width: 100vw;
}

// From codepen
#rete {
  height: 100% !important;
}
#modules {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 5;
}
.module-list {
  padding: 5px;
  cursor: pointer;
}
.module:hover {
  color: #a167e7;
}
.node .socket.number {
  background: #96b38a;
}
.node .socket.float {
  background: red;
}
</style>
