<template>
  <div class="container text-center">
    <div class="row">
      <div class="col">
        <h1> Matéria </h1>
        <select class="form-select form-select-lg mb-3" v-model="selectedMateria">
          <option v-for="(subject, index) in subjects" :key="subject.id" :value="index">{{ subject.name }}</option>
        </select>
      </div>

      <div class="col">
        <div v-if="selectedMateria !== undefined">
        <h1> Tópico </h1>
        <select  class="form-select form-select-lg mb-3" v-model="selectedTopic">
          <option v-for="(subject, index) in subjects[selectedMateria].children" :key="index" :value="subject">{{ subject.name }}</option>
        </select>
      </div>
      </div>
    </div>
  </div>

  <div class="container my-5">
    <div class="input-group">
      <span class="input-group-text">Conteudo</span>
      <textarea class="form-control" v-model="content"></textarea>
    </div>
  </div>
  <div class="container text-center">
    <button v-if="!contentGenerated" type="button" class="btn btn-primary" @click="this.handleContentSubmit()">Gerar Explicações</button>
    <div v-if="isLoading">
      <LoadingComponent />
    </div>
  </div>

  <div class="row" v-if="contentGenerated">
    <div class="col" v-for="(content, index) in this.generatedContent" :key="content.id" :value="content">
        <div class="card">
          <div class="card-header bg-success text-white" :class="{
            'bg-danger': content.status === 'rejected',
            'bg-warning': content.status === 'pending',
          }">
            {{contentTranslation[content.level]}}
          </div>
          <div class="card-body">
            <p class="card-text">{{content.content}}</p>
            <button class="btn btn-success mx-1" @click="this.handleContentAprove(content.id, 'approved', index)">Aprovar</button>
            <button class="btn btn-danger mx-1" @click="this.handleContentAprove(content.id, 'rejected', index)">Rejeitar</button>
          </div>
        </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import LoadingComponent from './LoadingComponent.vue';

export default {
data() {
  return {
    subjects: null,
    selectedMateria: undefined,
    selectedTopic: '',
    contentGenerated: false,
    isLoading: false,
    content: '',
    contentTranslation: {
      'simplified': 'Simplificado',
      'expanded': 'Outro ponto de vista'
    },
    generatedContent: '',
  };
},
components: {
  LoadingComponent,
},
methods: {
  setSubjects(subjects) {
    this.subjects = subjects;
  },
  handleContentSubmit() {
    this.contentGenerated = true;
    this.isLoading = true;
    axios
      .post('http://localhost:3000/create_content', {
        subject_id: this.selectedTopic.id,
        content: this.content
      })
      .then(response => {
        this.isLoading = false;
        console.log(response.data);
        this.generatedContent = response.data.contentResults;
      })
      .catch(e => {
        console.log(e)
      })
  },

  handleContentAprove(conentId, status, index) {
    axios
      .post('http://localhost:3000/submit_content_review', {
        content_result_id: conentId,
        status: status
      })
      .then(response => {
        this.generatedContent[index].status = status;
        console.log(response.data);
      })
      .catch(e => {
        console.log(e)
      })
  },
},
mounted () {
    axios
      .get('http://localhost:3000/list_subjects')
      .then(response => {
        this.setSubjects(response.data)
      })
      .catch(e => {
        console.log(e)
      })
  }
};
</script>

<style>
</style>
