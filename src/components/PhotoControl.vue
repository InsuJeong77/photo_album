<template>
  <v-container>
    <h1>파일 리스트</h1>
    <div v-for="(file,index) in fileList" :key="file.Key">
      <img style="width:128px;height:128px;" :src="viewImage(file.Key)">
      #{{index + 1}} {{file.Key}}
      <v-btn @click="deleteFile(file.Key)" color="red" icon>X</v-btn>
    </div>
    <h1>파일 업로더</h1>
    <input id="file-selector" ref="file" type="file" @change="handleFileUpload()">
    <v-btn @click="uploadFile" color="primary">업로드</v-btn>
  </v-container>
</template>

<script>
import { defineComponent } from '@vue/composition-api'
import AWS from 'aws-sdk'

export default defineComponent({
  data() {
    return {
      file: null,
      albumBucketName : "isjeongphoto",
      bucketRegion : "us-east-1",
      IdentityPoolId : "us-east-1:d8040b79-3e72-4b7d-a214-7a567c499f84",
      fileList : [],
      contentsAndUrl : [],
      photoUrlList : [],
    }
  },
  created() {
    this.getFiles()
  },
  methods:{
    handleFileUpload() {
      this.file = this.$refs.file.files[0]
      console.log(this.file, '파일이 업로드 되었습니다.')
    },
    // 파일을 업로드 하는 함수
    uploadFile() {
      AWS.config.update({
        region: this.bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: this.IdentityPoolId
        })
      });

      // var s3 = new AWS.S3({
      //   apiVersion: "2006-03-01",
      //   params: { Bucket: this.albumBucketName }
      // });

      let photoKey = this.file.name
      var upload = new AWS.S3.ManagedUpload({
        params: {
          Bucket: this.albumBucketName,
          Key: photoKey,
          Body: this.file
        }
      });
      var promise = upload.promise();
    
      promise.then(
        function(data) {
          alert("Successfully uploaded photo.");
          this.getFiles()
          console.log(data)
        },
        function(err) {
          return alert("There was an error uploading your photo: ", err.message);
        }
      );
    },
    // 파일을 가져오는 함수
    getFiles() {
      AWS.config.update({
        region: this.bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: this.IdentityPoolId
        })
      });

      var s3 = new AWS.S3({
        apiVersion: "2006-03-01",
        params: { Bucket: this.albumBucketName }
      });

      s3.listObjectsV2({ Delimiter: "/" }, (err, data) => {
        if (err) {
          return alert("There was an error listing your albums: " + err.message);
        } else {
          // for(let i=0; i<data.Contents.length ;i++ ) {
          //   var key = data.Contents[i][0].name;
          //   var photoUrl = encodeURIComponent(key) + '/'
          //   data.Contents[i].push(photoUrl);
          // }
          this.fileList = data.Contents;
          console.log(data)
        }
      });
    },
    // key 값에 해당하는 이미지 파일을 보여준다.
    viewImage(key) {
      AWS.config.update({
        region: this.bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: this.IdentityPoolId
        })
      });

      var s3 = new AWS.S3({
        apiVersion: "2006-03-01",
        params: { Bucket: this.albumBucketName }
      });

      console.log(s3.endpoint);
      // let photoKey = this.file.name
      
      s3.listObjectsV2({ Delimiter: "/" }, (err) => {
        if (err) {
          return alert("There was an error viewing your album: " + err.message);
        }
        var href = s3.endpoint.href;
        var bucketUrl = href + this.albumBucketName + "/";
        var photoUrl = bucketUrl + encodeURIComponent(key) + '/'
        console.log(photoUrl)
        return photoUrl
      }); 
      return alert("viewImage unexpected error : ");
    },
    // 파일을 지운다.
    deleteFile(key) {
      AWS.config.update({
        region: this.bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: this.IdentityPoolId
        })
      });

      var s3 = new AWS.S3({
        apiVersion: "2006-03-01",
        params: { Bucket: this.albumBucketName }
      });

      s3.deleteObject({ Key: key }, (err) => {
        if (err) {
          return alert("There was an error deleting your photo: ", err.message);
        }
        alert("Successfully deleted photo.");
        this.getFiles()
      });
    }
  }
})
</script>
