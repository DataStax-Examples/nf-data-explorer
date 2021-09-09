<template>
  <el-container class="form-container">
    <el-card class="form-card">
      <h3>Astra DB + Netflix Data Explorer</h3>
      <p>
        Upload your secure connect bundle (find it using the docs&nbsp;
        <a
          href="https://docs.datastax.com/en/astra/docs/obtaining-database-credentials.html"
          target="_blank"
          >here</a
        >) below and add an application token (generate one using the docs&nbsp;
        <a
          href="https://docs.datastax.com/en/astra/docs/manage-application-tokens.html"
          target="_blank"
          >here</a
        >) to connect to your Astra DB instance.
      </p>
      <br />
      <el-form ref="form" :model="astraDBForm" label-width="120px">
        <el-form-item label="Connect Bundle">
          <el-upload
            :on-change="handleChange"
            class="upload-demo"
            drag
            action="https://jsonplaceholder.typicode.com/posts/"
            :file-list="fileList"
            :multiple="false"
          >
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">
              Drop file here or <em>click to upload</em>
            </div>
            <div slot="tip" class="el-upload__tip">
              connection bundle zip files can be found on the connect page of
              your Astra DB instance.
            </div>
          </el-upload>
        </el-form-item>
        <el-form-item label="Application Token">
          <el-input v-model="astraDBForm.applicationToken"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" :disabled="saveInProgress" @click="submit"
            >Connect</el-button
          >
        </el-form-item>
      </el-form>
    </el-card>
    <br />
    <p>
      Check out the nf-data-explorer repository
      <a href="https://github.com/Netflix/nf-data-explorer" target="_blank"
        >here</a
      >.
    </p>
  </el-container>
</template>
<script lang="ts">
import Vue from 'vue';
import {
  Button,
  Container,
  Form,
  FormItem,
  Input,
  Upload,
  Card,
} from 'element-ui';
import router from '@/router';
import { insertAstraDBConfig } from '@/services/AstraDBService';
import { notify } from '@/utils/message-utils';
import { NotificationType } from '@/typings/notifications';
export default Vue.extend({
  name: 'ConnectionManagerView',
  components: {
    [Button.name]: Button,
    [Container.name]: Container,
    [Card.name]: Card,
    [Form.name]: Form,
    [FormItem.name]: FormItem,
    [Upload.name]: Upload,
    [Input.name]: Input,
  },
  data() {
    return {
      saveInProgress: false,
      fileList: [],
      rawFile: null,
      astraDBForm: {
        applicationToken: '',
      },
    };
  },
  methods: {
    handleChange(file: any) {
      this.rawFile = file.raw;
    },
    async submit() {
      this.saveInProgress = true;
      try {
        if (this.astraDBForm.applicationToken && this.rawFile) {
          await insertAstraDBConfig(
            this.astraDBForm.applicationToken,
            this.rawFile,
          );
          notify(
            NotificationType.Success,
            'Successfully added Astra DB Config',
            'AstraDB Config added.',
          );
          router.push(
            '/cassandra/clusters/astra/query?view=schema&query=&keyspace=&table=',
          );
        } else {
          throw new Error('Please fill out all fields');
        }
      } catch (err) {
        notify(NotificationType.Error, err.title, err.message);
      } finally {
        this.saveInProgress = false;
      }
    },
  },
});
</script>
<style module></style>
<style>
.form-container {
  background-color: #f3f5f8;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}
.form-card {
  width: 546px;
  padding: 20px;
}
</style>
