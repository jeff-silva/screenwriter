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
    <app-content
      v-model="editor.fileData.title"
      style="border: solid 2px red"
    />
    <pre>editor: {{ editor }}</pre>
  </div>
</template>

<script setup>
import _ from "lodash";
import { useFileSystemAccess } from "@vueuse/core";

class EditorFileData {
  constructor(data = {}) {
    this.update(data);
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
      line.type = line.type || "action";
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

editor.fileData.update({
  title: "Wizard of Oz",
  authors: [{ name: "Frank Baum", email: null }],
  script: [
    { type: "header", text: "EXT. FOREST. DAY" },
    {
      type: "action",
      value: `DOROTHY, TIN MAN, SCARECROW and TOTO walk through a thick forest in the Land of Oz. Dorothy carries a basket, the Tin Man carries an axe and an oil can. The road is paved with yellow brick and is covered with dried branches and dead leaves. The Emerald City is seen far in the distance.`,
    },
    {
      type: "character",
      value: "DOROTHY",
    },
    {
      type: "dialog",
      value: "It's really scary in these woods!",
    },
    {
      type: "action",
      value: "They hear a deep growl from wild animals in the trees!",
    },
    {
      type: "character",
      value: "DOROTHY",
    },
    {
      type: "dialog",
      value: "What was that?",
    },
    {
      type: "character",
      value: "SCARECROW",
    },
    {
      type: "dialog",
      value: "Hopefully not a beast who likes to eat straw!",
    },
    {
      type: "action",
      value: "Toto stands at attention with his ears perked up.",
    },
    {
      type: "character",
      value: "DOROTHY",
    },
    {
      type: "dialog",
      value:
        "How much longer before we are out of this forest Tin Man? I don't like it here!",
    },
    {
      type: "character",
      value: "TIN MAN",
    },
    {
      type: "dialog",
      value: `I really can't say. I've never been to the Emerald City. My Father told me that it is a long and dangerous journey! I am not worried because I have my oil can for my joints. Besides, you have the kiss from the Good Witch. That will protect us from harm!`,
    },
    {
      type: "character",
      value: "DOROTHY",
    },
    {
      type: "dialog",
      value: `What about Toto? We have to protect him.`,
    },
    {
      type: "action",
      value: `Toto starts to bark and growl at a tree in the forest. We hear a loud roar and from behind a tree leaps a large LION. The Scarecrow, Tin Man and Dorothy hide behind each other while Toto fearlessly starts to nip at the lion. The lion picks up Toto with his large teeth and shakes him. Dorothy slaps the lion across the nose. The lion drops Toto.`,
    },
    {
      type: "character",
      value: "DOROTHY",
    },
    {
      type: "dialog",
      value: `You should be ashamed of yourself! Why don't you pick on someone your own size!`,
    },
    {
      type: "character",
      value: "LION",
    },
    {
      type: "dialog",
      value: `Ouch, that smarts. What did you do that for? I didn't hurt him!`,
    },
    {
      type: "character",
      value: "DOROTHY",
    },
    {
      type: "dialog",
      value: `Nobody hurts Toto!`,
    },
    {
      type: "action",
      value: `Dorothy picks up Toto and holds him in her arms.`,
    },
    {
      type: "character",
      value: "LION",
    },
    {
      type: "dialog",
      value: `Well, you didn't have to slap me did you? Am I bleeding?`,
    },
    {
      type: "character",
      value: "DOROTHY",
    },
    {
      type: "dialog",
      value: `No, you are not bleeding you big coward!`,
    },
    {
      type: "character",
      value: "LION",
      observation: "ashamed",
    },
    {
      type: "dialog",
      value: `Yes, you are right -- I don't have any courage at all. I am nothing but a big coward. I've always been a coward. I guess I was born`,
    },
    {
      type: "action",
      value: `The Cowardly Lion starts to cry.`,
    },
  ],
});
</script>
