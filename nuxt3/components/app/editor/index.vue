<template>
  <div>
    <div class="d-flex ga-2">
      <v-btn
        text="Abrir"
        @click="editor.fileOpen()"
      />
      <v-btn
        text="Novo"
        @click="editor.fileCreate()"
      />
      <v-btn
        text="Salvar"
        @click="editor.fileSave()"
      />
    </div>

    <div style="display: none">
      <app-editor-script-action />
      <app-editor-script-character />
      <app-editor-script-dialog />
      <app-editor-script-header />
    </div>

    <v-container>
      <div class="d-flex justify-center">
        <div>{{ editor.fileData.title }}</div>
      </div>
      <div
        class="d-flex flex-column ga-2"
        style="font-family: monospace"
      >
        <template v-for="o in editor.fileData.script">
          <div class="bg-red">{{ o.value }}</div>
          <component :is="appEditorScriptAction" />
          <!-- <component v-bind="o" /> -->
          <pre>{{ o }}</pre>
        </template>
      </div>

      <pre>editor.fileData.script: {{ editor.fileData.script }}</pre>
    </v-container>
    <!-- <v-text-field v-model="editor.fileData.title" /> -->
    <!-- <app-content
      v-model="editor.fileData.title"
      style="border: solid 2px red"
    /> -->
    <!-- <pre>editor: {{ editor }}</pre> -->
  </div>
</template>

<script setup>
import _ from "lodash";
import { useFileSystemAccess } from "@vueuse/core";

import appEditorScriptAction from "@/components/app/editor/script/action.vue";

class EditorFileData {
  constructor(data = {}) {
    this.update(data);

    // const modules = import.meta.glob("./*.vue", { eager: true });
    // console.clear();
    // console.log(modules, Object.entries(modules));
  }

  update(data = {}) {
    this.title = data.title || null;

    this.copyright = _.merge(data.copyright || {}, {
      holder: "",
      year: "",
    });

    this.authors = data.authors || [];
    this.script = data.script || [];
    this.characters = data.characters || [];

    this.scriptSync();
    this.charactersSync();
  }

  scriptSync() {
    this.script.map((line) => {
      line.is = line.is || "app-editor-script-action";
      line.value = line.value || "";
      line.observation = line.observation || "";
    });
  }

  charactersSync() {
    let characters = {};
    this.script
      .filter((line) => line.type == "character")
      .map((line) => {
        const uniqueName = line.value;
        characters[uniqueName] = { name: line.value };
      });
    this.characters = Object.values(characters);
  }
}

const editor = reactive({
  file: useFileSystemAccess(),
  fileOptions: {
    excludeAcceptAllOption: false,
    multiple: false,
    types: [
      {
        description: "ScreenWriter file",
        accept: {
          "text/*": [".scrw"],
        },
      },
    ],
  },
  fileData: new EditorFileData(),
  contentTypes: [
    {
      id: "header",
      name: "Scene Header",
    },
    {
      id: "action",
      name: "Action",
    },
    {
      id: "character",
      name: "Character",
    },
    {
      id: "dialog",
      name: "Dialog",
    },
  ],
  async fileCreate() {
    await editor.file.create(editor.fileOptions);
  },
  async fileOpen() {
    await editor.file.open(editor.fileOptions);
    editor.fileData = JSON.parse(editor.file.data || "{}");
  },
  async fileSave() {
    editor.file.data = JSON.stringify(editor.fileData);
    editor.file.save();
  },
});

import wizardOfOz from "@/script-examples/wizard-of-oz.js";
editor.fileData.update(wizardOfOz);
</script>
