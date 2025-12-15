<script setup>
import { ref } from 'vue'
import {
  DialogRoot,
  DialogTrigger,
  DialogPortal,
  DialogOverlay,
  DialogContent,
  DialogTitle,
  DialogDescription,
  DialogClose,
} from 'reka-ui'

// TinyMCE imports
import 'tinymce/tinymce'
import 'tinymce/icons/default'
import 'tinymce/themes/silver'
import 'tinymce/plugins/link'
import 'tinymce/models/dom'
import 'tinymce/skins/ui/oxide/skin.min.css'
import Editor from '@tinymce/tinymce-vue'

const isOpen = ref(false)
const editorContent = ref('<p>Click the link button in the toolbar to open TinyMCE\'s link dialog.</p>')
</script>

<template>
  <div class="container">
    <h1>Reka-UI + TinyMCE Focus Issue Reproduction</h1>

    <p>This demonstrates the focus trap issue when TinyMCE dialogs open inside reka-ui modals.</p>

    <h2>Steps to reproduce:</h2>
    <ol>
      <li>Click "Open Modal" button</li>
      <li>In the TinyMCE editor, click the "Link" button in the toolbar</li>
      <li>Try to type in the URL input field - <strong>it won't work</strong></li>
    </ol>

    <h2>Expected behavior:</h2>
    <p>The TinyMCE link dialog inputs should be focusable and typeable.</p>

    <h2>Actual behavior:</h2>
    <p>Focus is immediately returned to the modal, making TinyMCE dialog inputs appear disabled.</p>

    <hr />

    <DialogRoot v-model:open="isOpen">
      <DialogTrigger class="button">
        Open Modal
      </DialogTrigger>

      <DialogPortal>
        <DialogOverlay class="overlay" />
        <DialogContent class="content">
          <DialogTitle class="title">Modal with TinyMCE Editor</DialogTitle>
          <DialogDescription class="description">
            Try clicking the Link button in the toolbar below, then type in the URL field.
          </DialogDescription>

          <div class="editor-container">
            <Editor
              v-model="editorContent"
              license-key="gpl"
              :init="{
                height: 300,
                menubar: false,
                plugins: 'link',
                toolbar: 'bold italic | link',
                skin: false,
                content_css: false,
                license_key: 'gpl',
              }"
            />
          </div>

          <div class="buttons">
            <DialogClose class="button">Close</DialogClose>
          </div>
        </DialogContent>
      </DialogPortal>
    </DialogRoot>

    <hr />

    <h2>Root Cause:</h2>
    <p>
      Reka-ui's <code>FocusScope</code> component uses document-level <code>focusin</code>/<code>focusout</code>
      event listeners to trap focus within the modal. When TinyMCE opens its dialog (which renders in
      <code>.tox-tinymce-aux</code> outside the modal container), FocusScope detects focus leaving
      and immediately refocuses back to the modal.
    </p>

    <h2>Potential Solution:</h2>
    <p>
      Export <code>createFocusScopesStack</code> from reka-ui so external code can programmatically
      add/remove focus scopes. This would allow pausing the parent modal's focus trap when TinyMCE
      (or other third-party) dialogs open.
    </p>
  </div>
</template>

<style>
/* Import TinyMCE content CSS */
@import 'tinymce/skins/content/default/content.min.css';

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
  font-family: system-ui, -apple-system, sans-serif;
}

.button {
  background: #4f46e5;
  color: white;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.375rem;
  cursor: pointer;
  font-size: 1rem;
}

.button:hover {
  background: #4338ca;
}

.overlay {
  background: rgba(0, 0, 0, 0.5);
  position: fixed;
  inset: 0;
  z-index: 50;
}

.content {
  background: white;
  border-radius: 0.5rem;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 90vw;
  max-width: 600px;
  max-height: 85vh;
  padding: 1.5rem;
  z-index: 51;
  overflow: auto;
}

.title {
  margin: 0 0 0.5rem;
  font-size: 1.25rem;
  font-weight: 600;
}

.description {
  margin: 0 0 1rem;
  color: #6b7280;
}

.editor-container {
  margin-bottom: 1rem;
}

.buttons {
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
}

code {
  background: #f3f4f6;
  padding: 0.125rem 0.25rem;
  border-radius: 0.25rem;
  font-size: 0.875em;
}

h2 {
  margin-top: 1.5rem;
}

ol {
  padding-left: 1.5rem;
}

li {
  margin-bottom: 0.5rem;
}
</style>
