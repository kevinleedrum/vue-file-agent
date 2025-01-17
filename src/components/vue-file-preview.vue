<template>
  <div :class="['file-preview-wrapper-' + fileData.ext(), fileData.isImage() ? 'file-preview-wrapper-image' : 'file-preview-wrapper-other', 'file-category-' + fileData.icon().category, {'file-is-playing-av': fileData.isPlayingAv}, {'is-deletable': deletable === true}, {'is-editable': editable === true}, {'is-edit-input-focused': isEditInputFocused}, {'has-error': fileData.error}]">
    <div class="file-error-wrapper" v-if="fileData.error">
      <div class="file-error-message file-error-message-client" v-if="fileData.error">
        {{ fileData.getErrorMessage(errorText) }}
      </div>
    </div>
    <div ref="wrapper" class="file-av-wrapper" v-if="fileData.isPlayableAudio() || fileData.isPlayableVideo()">
        <div class="file-av-action" @click="playAv(fileData)">
          <span class="file-av-stop">
            <svg width="24" height="24" viewBox="0 0 24 24"><path d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/><path d="M0 0h24v24H0z" fill="none"/></svg>
          </span>
          <span class="file-av-play">
            <svg width="48" height="48" viewBox="0 0 48 48"><path d="M24 4C12.95 4 4 12.95 4 24s8.95 20 20 20 20-8.95 20-20S35.05 4 24 4zm-4 29V15l12 9-12 9z"></path></svg>
          </span>
        </div>
    </div>
    <span class="file-preview" v-bind:class="{'image-preview': fileData.isImage(), 'other-preview': !fileData.isImage(), 'dark-content': fileData.isImage() && fileData.isDarkColor()}" v-bind:style="{'background-color': fileData.color(), 'background-imagex': 'url(' + fileData.src() + ')', widthx: fileData.width + 'px', heightx: fileData.height + 'px'}">
      <span class="thumbnail" style="position: absolute; top: 0; right: 0; bottom: 0; left: 0; overflow: hidden;">
        <img v-if="fileData.isImage() || fileData.isPlayableVideo()" class="file-preview-img" v-bind:src="fileData.src()">
      </span>
      <span class="file-ext">{{ fileData.ext() }}</span>
      <span class="file-size">{{ fileData.size() }}</span>
      <span v-if="deletable" class="file-delete" @click="removeFileData(fileData)" @touchstart="filenameClearPressed()" @mousedown="filenameClearPressed()" >
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"><path d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/><path d="M0 0h24v24H0z" fill="none"/></svg>
      </span>
      <span class="file-name" @click="editFileName()">
        <input class="file-name-input" ref="input" v-if="editable === true" :disabled="disabled === true" type="text" :value="fileData.name(true)" @focus="editInputFocused()" @blur="editInputBlured()" @change="filenameChanged()" @input="filenameChanged()" @keyup.enter="filenameChanged(true)" @keyup.esc="filenameChanged(false)">
        <span class="file-name-edit-icon" v-if="editable === true">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/><path d="M0 0h24v24H0z" fill="none"/></svg>
        </span>
        <span class="file-name-text">{{ fileData.name(true) }}</span>
      </span>
      <span v-if="fileData.dimensions.width && fileData.dimensions.height" class="image-dimension">
        <span class="image-dimension-width">{{ fileData.dimensions.width }}</span><span class="image-dimension-height">{{ fileData.dimensions.height }}</span>
      </span>
      <span class="file-progress" v-if="fileData.hasProgress()" v-bind:class="{'file-progress-full': fileData.progress() >= 100, 'has-file-progress': fileData.progress() > 0}">
        <span class="file-progress-bar" v-bind:style="{width: fileData.progress() + '%'}"></span>
      </span>
      <span class="file-icon">
        <VueFileIcon :ext="fileData.ext()"></VueFileIcon>
      </span>
    </span>
  </div>
</template>
<script>

  import utils from '../lib/utils';
  import VueFileIcon from './vue-file-icon.vue';

  export default {
    props: ['fileData', 'deletable', 'editable', 'errorText', 'disabled'],
    components: {
      VueFileIcon
    },
    data: function(){
      return {
        isEditInputFocused: false,
        isEditCancelable: true,
      }
    },
    methods: {

      createThumbnail(fileData, video){
        var canvas = document.createElement('canvas');
        utils.createVideoThumbnail(video, canvas, this.fileData.thumbnailSize).then((thumbnail) => {
          fileData.imageColor = thumbnail.color;
          fileData.videoThumbnail = thumbnail.url;
          fileData.dimensions.width = thumbnail.width;
          fileData.dimensions.height = thumbnail.height;
        });
      },

      playAv(fileData){
        if(fileData.stopAv){
          fileData.stopAv();
          return;
        }
        var createObjectURL = (window.URL || window['webkitURL'] || {}).createObjectURL;
        var revokeObjectURL = (window.URL || window['webkitURL'] || {}).revokeObjectURL;

        var wrapper = this.$refs.wrapper;
        var player = document.createElement(fileData.isAudio() ? 'audio' : 'video');
        if(fileData.isPlayableVideo()){
          this.createThumbnail(fileData, player);
          player.poster = fileData.src();
        }
        player.controls = true;
        // player.style.width = this.prvWidth + 'px';
        wrapper.appendChild(player);
        var url = fileData.url || createObjectURL(fileData.file); 
        player.src = url; 
        player.play();
        fileData.isPlayingAv = true;
        fileData.stopAv = function(){
          player.src = null;
          wrapper.removeChild(player);
          revokeObjectURL(url);
          fileData.isPlayingAv = false;
          fileData.stopAv = null;
        }
      },

      removeFileData(fileData){
        if(this.clearFilename()){
          return;
        }
        if(this.disabled === true){
          return;
        }
        this.$emit('remove', fileData);
      },

      editFileName(){
        if(this.editable !== true){
          return;
        }
        if(!this.$refs.input){
          return;
        }
        this.$refs.input.focus();
      },

      editInputFocused(){
        this.isEditInputFocused = true;
        this.isEditCancelable = true;
      },

      editInputBlured(){
        var value = this.$refs.input.value;
        this.fileData.customName = value;
        var timeout = 100;
        setTimeout(()=> {
          this.$nextTick(()=> {
            if(!this.isEditCancelable){
              return;
            }
            this.isEditInputFocused = false;
          });
        }, timeout);
      },

      filenameChanged(completed){
        if(completed){
          this.$refs.input.blur();
        }
        if(completed === false){
          this.clearFilename();
        }
      },

      filenameClearPressed(){
        if(!(this.editable === true && this.isEditInputFocused)){
          return;
        }
        this.isEditCancelable = false;
      },

      clearFilename(){
        if(!(this.editable === true && this.isEditInputFocused)){
          return false;
        }
        this.$refs.input.value = '';
        this.isEditCancelable = true;
        this.editInputBlured();
        return true;
      },

    },
  }
</script>