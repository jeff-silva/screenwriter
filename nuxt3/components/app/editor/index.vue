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
    <v-text-field v-model="editor.fileData.title" />
    <pre>editor: {{ editor }}</pre>
  </div>
</template>

<script setup>
import { useFileSystemAccess } from "@vueuse/core";

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
  fileData: {
    title: null,
    authors: [
      {
        name: null,
        email: null,
      },
    ],
    content: [],
  },
  contentTypes: [
    {
      id: "sceneHeader",
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
</script>
