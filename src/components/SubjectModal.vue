<template>
    <BaseModal ref="modalRef" id="subjectAddOrEditModal"
      :title="isAdd ? 'Añadiendo nueva asignatura' : 'Editando asignatura'" 
      :name="name">
      <template #body>
        <form id="addOrEditSubjectForm" 
          @submit.prevent="e => setSubject()">
          <div class="container">
            <TextBox :start="subject.name" id="e-name" label="Name" required="true" 
              @change="(v) => name=v"/>
            <TextBox :start="subject.short" id="e-short" label="Acrónimo" required="true"/>
            <br>
            <TextBox :start="subject.degree" id="e-degree" label="Grados" required="true"/>
            <TextBox :start="subject.codes" id="e-codes" label="Códigos GEA" required="true"/>
            <div class="form-group row g-1 form-inline align-items-baseline">
              <div class="col-3 text-end">
                <label for="e-semester">Cuatrimestre</label>
             </div>
             <div class="col-auto">
              <select id="e-semester" name="e-semester" v-model="semester" class="form-control">
                <option value="" disabled>Selecciona un cuatrimestre</option>
              <option value="SPRING">C2</option>
              <option value="FALL">C1</option>
            </select>
        </div>
          </div>
            <TextBox :start="''+subject.credits" id="e-credits" label="Créditos" required="true" pattern="\d{1,2}" />
            <br>
          </div>
          <button type="submit" class="invisible">Submit</button>
        </form>
      </template>
      <template #footer>
        <button @click.prevent="() => setSubject()" class="btn btn-primary">
          {{ isAdd ? 'Añadir asignatura' : 'Confirmar cambios a' }}
          <span class="name">{{ name }}</span>
        </button>
      </template>
    </BaseModal>
</template>

<script setup>

import BaseModal from './BaseModal.vue';
import TextBox from './TextBox.vue'

import { gState } from '../state.js';
import { ref } from 'vue'

const emit = defineEmits(['add', 'edit'])

const props = defineProps({
  subject: Object,
  isAdd: Boolean, // otherwise, editing existing instead of adding
})

let modalRef = ref(null);
let name = ref(props.subject.name);
let semester = ref(props.subject.semester || "");

function setSubject() {    
  const subject = props.subject;
  const form = document.getElementById("addOrEditSubjectForm")
  const valueFor = (name) => {
    const input = form.querySelector(`input[name=${name}]`)
    if (!input) console.log("ERROR: no input for name", name, "in", form)
    return input.value
  }

  console.log("saving subject...", subject, form)

  // comprueba validez de todos los campos, y sobreescribe resultado
  if ( ! form.checkValidity()) {
    // fuerza a que se muestren los errores simulando un envío
    // (pero como hay errores, no se va a enviar nada :-)
    form.querySelector("button[type=submit]").click()
    return; 
  }    

  // todo válido: lanza evento a padre, y cierra modal
  emit(props.isAdd ? 'add' : 'edit', new gState.model.Subject(subject.id,
    valueFor("e-name"), 
    valueFor("e-short"), 
    valueFor("e-degree"), 
    +valueFor("e-credits"),
    semester.value,
    valueFor("e-codes"),
    subject.groups,
  ))
  modalRef.value.hide()    
}

// para que el padre pueda llamar a show (hide no debería hacer falta)
function show() {
  modalRef.value.show();
}
defineExpose({ show });
</script>

<style scoped>
    .name {
        font-weight: 200%;
    }
</style>