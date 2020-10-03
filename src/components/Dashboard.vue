<script>
import firebase from 'firebase';

export default {
  name: 'Dashboard',
  data() {
      return {
        filmOrder: {
          field: "",
          order: ""
        },
        photoOrder: {
          field: "",
          order: ""
        },
        homeActive: true,
        reelWindowActive: false,
        filmWindowActive: false,
        photoWindowActive: false,
        settingsWindowActive: false,
        screengrabWindowActive: false,
        showPhotos: true,
        updateActive: false,
        focus: 0,
        auth: '',
        uploadProgress: 0,
        screengrabUploadProgress: 0,
        progressBuffer: [0,0],
        newBio: {
          timestamp: "",
          photo: "",
          text: ""
        },
        items: [],
        stagedItems: [], //array of staged objects
        //avgProgress: 0,
        uploadDone: false,
        photoCount: 1,
        photo: {
          src: "",
          tag: "",
          thumbSrc100: "",
          thumbSrc350: "",
          timestamp: ""
        },
        reel: {
          finalFilm: ""
        },
        photos: [],
        filmReel: "",
        item: {
          title: "",
          genre: "",
          summary: "",
          finalFilm: "",
          credits: [
              {
                  role: "",
                  name: ""
              }
          ],
          screenGrabs: [
              ""
          ],
          timestamp: "",
          startTime: 0
        }
      }
    },
    beforeCreate() {
      console.log('before create hook');
    },
    created() {
      console.log('post authentication');
    },
    mounted() {
      console.log('mounted');
      this.setBio();
      this.setReel();
      this.setItems();
      this.setPhotos();
    },
    methods: {
        setPhotos() {
            firebase.firestore().collection('photos').orderBy('timestamp','desc').get().then((docs) => {
                docs.forEach((doc) => {
                  this.photos.push(doc.data());
                });
            });
        },
        setBio() {
            firebase.firestore().collection('bio').doc("bio").get().then((doc) => {
                this.newBio.text = doc.data().text;
                this.newBio.photo = doc.data().photo;
            });
        },
        async setReel() {
          console.log('setting reel');

          await firebase.firestore().collection("filmReel").doc('reel').get().then((doc) => {
            this.filmReel = doc.data().finalFilm;
            console.log("reel location: ", this.filmReel);
          });
        },
        setItems() {
            firebase.firestore().collection('portfolioItems').get().then((collection) => {
                //console.log(collection.docs);
                for(let i = 0; i < collection.docs.length; i++) {
                    //console.log(collection.docs[i].data().title);
                
                    //----- adding in dynamic data for project update forms -----

                    this.stagedItems.push(collection.docs[i].data());
                    //this.updateActive.push(collection.docs[i].data().title);
                    //this.updateActive[i].push();
                }

                console.log("ITEMS: " + this.stagedItems);
            });
        },
        add() {
            console.log('clicked')
            console.log(this.items)
            firebase.firestore().collection('portfolioItems').add(this.item).then(() => {
                this.item.title = "",
                this.item.genre = "",
                this.item.summary = "",
                this.item.finalFilm = "",
                this.item.credits = [
                    {
                        role: "",
                        name: ""
                    }
                ],
                this.item.screenGrabs = [
                    ""
                ],
                this.item.timestamp = Date.now();

                alert('Added photo');

                window.location.reload();

            })
        },
        addReel() {
            console.log(this.items)
            firebase.firestore().collection('filmReel').doc("reel").set(this.reel).then(()=>{

                this.reel.finalFilm = ""

                //refresh page
                setTimeout(function() {
                    document.location.reload();
                });
            })
        },
        addPhoto() {
            firebase.firestore().collection('photos').add(this.photo).then(()=>{

                this.photo = {
                    src: "",
                    tag: "",
                    thumbSrc100: "",
                    thumbSrc350: "",
                    timestamp: ""
                }

            })
        },
        addBio() {
            console.log('adding new bio');

            this.newBio.timestamp = Date.now();

            firebase.firestore().collection('bio').doc("bio").set(this.newBio).then(()=>{
                alert("Profile updated");
                this.toggleHome();
            })
        },
        stageUpdate(index, e) {
            //this.updateActive = true;
            //console.log("READY TO UPDATE");
            //console.log(i);
            //console.log(this.stagedItems[index].title);

            console.log(e);

            document.getElementById("update" + index).style.opacity = 1;
            document.getElementById("update" + index).style.pointerEvents = "all";
        },
        handleUpdate(index, e) {
            //console.log("updated " + e.title + " to " + this.stagedItems[index].title);

            firebase.firestore().collection('portfolioItems').doc(e[".key"]).set(this.stagedItems[index]).then(()=>{

                //alert("Changes Saved");

                //refresh page
                setTimeout(function() {
                    document.location.reload();
                });
            })
        },
        removeProject(e) {
            console.log(e.screenGrabs);

            //deleting screenGrab directory in storage

            if(confirm("Are you sure you want to delete " + e.title + "?")) {
                //deleting item from database
                firebase.firestore().collection('portfolioItems').where('title','==',e.title).get.then((querySnapshot) => {
                    querySnapshot.forEach((doc) => {
                      doc.ref.delete();
                    });

                    alert('Project deleted');
                    window.location.reload();
                });
            }
            else {
                //alert("Project was not deleted");
            }

        /* ---------- TODO ---------- */

        /*for(let i = 0; i < e.screenGrabs.length; i++) {
            firebase.storage().ref().child(e.imgPaths[i]).delete().then(function() {
                console.log("Image deleted successfully");
            }).catch(function(error) {
                console.error("Image not deleted successfully");
            });
        }*/

        },
        async removePhoto(e) {
    
            //deleting screenGrab directory in storage

            console.log("target object: " + e);

            if(confirm("Are you sure you want to delete this photo?")) {
                console.clear();
                console.log('attempting to remove photo....');
                console.log('target: ', e);
                console.log('object key: ' , e.timestamp);
                //deleting item from database
                await firebase.firestore().collection('photos').where('timestamp','==',e.timestamp).get().then((querySnapshot) => {
                    querySnapshot.forEach((doc) => {
                      doc.ref.delete();
                    });

                    alert('Photo deleted');

                    window.location.reload();
                })  
            }
            else {
                //alert("Photo was not deleted");
            }

            //---------- TODO ----------

            //recursively remove files from storage

        },
        newCredit() {
            this.item.credits.push(
                {
                    role: "",
                    name: ""
                }
            );
        },
        handleNewScreenGrab(e) {
            console.log("...handling screengrab upload for " + e.target.id);

            //console.log(e.target.id.substring(10));

            var parentObj = this;
            var task = [];

            for(let i = 0; i < e.target.files.length; i++) {
                //push file into storage
                task.push(firebase.storage().ref(parentObj.stagedItems[e.target.id.substring(10)].title + "/" + e.target.files[i].name).put(e.target.files[i]));
            
                var avgProgress = 0;

                //update progress bar and handle onComplete
                task[i].on('state_changed', function progress(snapshot) {
                        var percentage = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;

                        parentObj.progressBuffer[1] = percentage;

                        if((i > 0)) {
                            avgProgress = parentObj.progressBuffer[1] - parentObj.progressBuffer[0];
                        } else {
                          console.log(avgProgress);
                        }

                        //when done, turn green
                        if (percentage == 100) {
                            parentObj.uploadDone = true;
                        }

                        parentObj.uploadProgress = percentage;

                        parentObj.progressBuffer[0] = percentage;

                        //uploader.value = percentage;

                        console.log(percentage);
                    },
                    function error(err) {
                        console.log(err);
                    },
                    function complete() {
                        //console.log(storageRef.child(imgPath).getDownloadURL().getResults());

                        task[i].snapshot.ref.getDownloadURL().then(
                            function(downloadURL) {
                                console.log('File available at: ' + downloadURL)
                                
                                parentObj.stagedItems[e.target.id.substring(10)].screenGrabs.push(downloadURL + "");

                                console.log(parentObj.stagedItems[e.target.id.substring(10)].screenGrabs);
                                //parentObj.photoCount++;
                                //console.log(parentObj.photo);
                            }
                        )
                        //console.log("screenGrabs: " + parentObj.item.screenGrabs);

                        document.getElementById("update" + e.target.id.substring(10)).style.opacity = 1;
                        document.getElementById("update" + e.target.id.substring(10)).style.pointerEvents = "all";

                    }
                )
            }
        },
        handleRemoveScreengrab(index, photoIndex) {
            console.log("current proj: " + index);
            console.log("Removing image " + photoIndex + " from " + this.stagedItems[index].title);
            console.log("image key: " + this.items[index][".key"]);

            if(confirm("Are you sure you want to delete this screengrab?")) {
                //deleting item from database
                this.stagedItems[index].screenGrabs.splice(photoIndex,1);

                document.getElementById("update" + index).style.opacity = 1;
                document.getElementById("update" + index).style.pointerEvents = "all";
            }
            else {
                //alert("Photo was not deleted");
            }

        },
        handleScreengrabUpload(e) {
            console.log("...handling screengrab upload for " + this.item.title);

            var parentObj = this;
            var task = [];

            for(let i = 0; i < e.target.files.length; i++) {
                //push file into storage
                task.push(firebase.storage().ref(parentObj.item.title + "/" + e.target.files[i].name).put(e.target.files[i]));
            
                var avgProgress = 0;

                //update progress bar and handle onComplete
                task[i].on('state_changed', function progress(snapshot) {
                        var percentage = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;

                        parentObj.progressBuffer[1] = percentage;

                        if((i > 0)) {
                            avgProgress = parentObj.progressBuffer[1] - parentObj.progressBuffer[0];
                        } else {
                          console.log(avgProgress);
                        }

                        //when done, turn green
                        if (percentage == 100) {
                            parentObj.uploadDone = true;
                        }

                        parentObj.screengrabUploadProgress = percentage;

                        parentObj.progressBuffer[0] = percentage;

                        //uploader.value = percentage;

                        console.log(percentage);
                    },
                    function error(err) {
                        console.log(err);
                    },
                    function complete() {
                        //console.log(storageRef.child(imgPath).getDownloadURL().getResults());

                        task[i].snapshot.ref.getDownloadURL().then(
                            function(downloadURL) {
                                console.log('File available at: ' + downloadURL)
                                
                                parentObj.item.screenGrabs.push(downloadURL + "");

                                console.log(parentObj.item.screenGrabs);
                                //parentObj.photoCount++;
                                //console.log(parentObj.photo);
                            }
                        )

                    }
                )
            }
        },
        handlePhotoUpload(e) {
            console.log("...handling photo upload");

            var parentObj = this;
            var task = [];

            for(let i = 0; i < e.target.files.length; i++) {
                //push file into storage
                task.push(firebase.storage().ref("Photos/" + e.target.files[i].name).put(e.target.files[i]));
            
                var avgProgress = 0;

                //update progress bar and handle onComplete
                task[i].on('state_changed', function progress(snapshot) {


                        var percentage = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;

                        parentObj.progressBuffer[1] = percentage;

                        if((i > 0)) {
                            avgProgress = parentObj.progressBuffer[1] - parentObj.progressBuffer[0];
                        } else {
                          console.log(avgProgress);
                        }

                        //when done, turn green
                        if (percentage == 100) {
                            parentObj.uploadDone = true;
                        }

                        parentObj.uploadProgress = percentage;

                        parentObj.progressBuffer[0] = percentage;

                        //uploader.value = percentage;

                        console.log(percentage);
                    },
                    function error(err) {
                        console.log(err);
                    },
                    function complete() {
                        //console.log(storageRef.child(imgPath).getDownloadURL().getResults());

                        task[i].snapshot.ref.getDownloadURL().then(
                            function(downloadURL) {
                                console.log('File available at: ' + downloadURL)

                                var fileName = downloadURL.slice(downloadURL.indexOf('%') + 3, downloadURL.indexOf('?'));
                                var  fileSlice = fileName.slice(0, fileName.indexOf('.'));

                                console.log('***File name: ' + fileName);
                                console.log('***File slice: ' + fileSlice);
                                
                                parentObj.photo = {
                                    src: downloadURL + "",
                                    tag: ":)",
                                    thumbSrc100: "" + downloadURL.replace(fileSlice, "thumbnails%2F" + fileSlice + "_100x100"),
                                    thumbSrc350: "" + downloadURL.replace(fileSlice, "thumbnails%2F" + fileSlice + "_350x350"),
                                    timestamp: Date.now()
                                }

                                console.log('Thumbnail available at: ' + parentObj.photo.thumbSrc100);

                                console.log(parentObj.photo);
                                parentObj.addPhoto();
                            }
                        )
                        //console.log("screenGrabs: " + parentObj.item.screenGrabs);
                    }
                )
            }
        },
        handleFilmUpload(e) {
            console.log("...handling film upload for " + this.item.title);

            var parentObj = this;

            var file = e.target.files[0];

            var filmPath = this.item.title + "/" + file.name;

            //create a storage ref
            var storageRef = firebase.storage().ref(filmPath);

            //upload file
            var task = storageRef.put(file);

            //update progress bar
            task.on('state_changed', 
            
                function progress(snapshot) {
                    var percentage = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
                    //uploader.value = percentage;
                    
                    parentObj.uploadProgress = percentage;

                    if(percentage == 100) {
                        //uploader.style.background = "#66BB6A";
                        parentObj.uploadDone = true;
                    }
                },
                function error(err) {
                    console.log(err);
                },
                function complete() {
                    //console.log(storageRef.child(imgPath).getDownloadURL().getResults());

                    task.snapshot.ref.getDownloadURL().then(
                        function(downloadURL) {
                            console.log('File available at: ' + downloadURL)
                            parentObj.item.finalFilm = downloadURL + ""
                        }
                    )

                    console.log("video: " + parentObj.item.screenGrabs);
                }
            )
        },

        handleReelUpload(e) {
            console.log("...handling reel upload for ");

            var parentObj = this;

            var file = e.target.files[0];

            var filmPath = "Reel/" + file.name;

            //create a storage ref
            var storageRef = firebase.storage().ref(filmPath);

            //upload file
            var task = storageRef.put(file);

            //update progress bar
            task.on('state_changed', 
            
                function progress(snapshot) {
                    var percentage = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
                    //uploader.value = percentage;

                    parentObj.uploadProgress = percentage;

                    if(percentage == 100) {
                        //uploader.style.background = "#66BB6A";
                        parentObj.uploadDone = true;
                    }
                },
                function error(err) {
                    console.log(err);
                },
                function complete() {
                    //console.log(storageRef.child(imgPath).getDownloadURL().getResults());

                    task.snapshot.ref.getDownloadURL().then(
                        function(downloadURL) {
                            console.log('File available at: ' + downloadURL)
                            parentObj.reel.finalFilm = downloadURL + ""
                        }
                    )
                }
            )
        },

        handleProfileUpload(e) {
            console.log("...handling profile upload");

            var parentObj = this;

            var file = e.target.files[0];

            var path = "Profile/" + file.name;

            //create a storage ref
            var storageRef = firebase.storage().ref(path);

            //upload file
            var task = storageRef.put(file);

            //update progress bar
            task.on('state_changed', 
            
                function progress(snapshot) {
                    var percentage = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
                    //uploader.value = percentage;

                    if(percentage == 100) {
                        //uploader.style.background = "#66BB6A";
                    }
                },
                function error(err) {
                    console.log(err);
                },
                function complete() {
                    //console.log(storageRef.child(imgPath).getDownloadURL().getResults());

                    task.snapshot.ref.getDownloadURL().then(
                        function(downloadURL) {
                            console.log('File available at: ' + downloadURL);
                            parentObj.newBio.photo = downloadURL + "";
                            e.target.parentElement.style.backgroundImage = "url(" + downloadURL + ")";
                        }
                    )
                }
            )
        },
        //----------COMPRESSION METHODS----------
        imageCompression() {

        },

        //----------NAVIGATION----------
        toggleHome() {
            //alert('Home');
            this.filmWindowActive = false;
            this.photoWindowActive = false;
            this.settingsWindowActive = false;
            this.reelWindowActive = false;
            this.screengrabWindowActive = false;
            this.homeActive = true;
            this.photoCount = 1;
            this.uploadDone = false;
            this.uploadProgress = 0;
            this.screengrabUploadProgress = 0;
            this.item.credits = [""];
        },
        toggleFilm() {
            //alert('Film section');
            if(this.filmWindowActive) {
                this.homeActive = true;
                console.log('home? ' + this.homeActive);
            }
            else {
                this.homeActive = false;
            }
            this.filmWindowActive = !this.filmWindowActive;
            this.settingsWindowActive = false;
            this.photoWindowActive = false;
            this.reelWindowActive = false;
            this.screengrabWindowActive = false;
        },
        toggleReel() {
            //alert('Film section');
            if(this.reelWindowActive) {
                this.homeActive = true;
                console.log('home? ' + this.homeActive);
            }
            else {
                this.homeActive = false;
            }
            this.reelWindowActive = !this.reelWindowActive;
            this.settingsWindowActive = false;
            this.photoWindowActive = false;
            this.filmWindowActive = false;
            this.screengrabWindowActive = false;
        },
        togglePhoto() {
            //alert('Photo section');
            if(this.photoWindowActive) {
                this.homeActive = true;
            }
            else {
                this.homeActive = false;
            }
            this.photoWindowActive = !this.photoWindowActive;
            this.settingsWindowActive = false;
            this.filmWindowActive = false;
            this.reelWindowActive = false;
            this.screengrabWindowActive = false;
        },
        toggleSettings() {
            //alert('Settings section');
            if(this.settingsWindowActive) {
                this.homeActive = true;
            }
            else {
                this.homeActive = false;
            }
            this.settingsWindowActive = !this.settingsWindowActive;
            this.photoWindowActive = false;
            this.filmWindowActive = false;
            this.reelWindowActive = false;
            this.screengrabWindowActive = false;
        },
        toggleScreengrab(index) {
            //alert('Settings section');
            if(this.screengrabWindowActive) {
                this.homeActive = true;
            }
            else {
                this.homeActive = false;
            }
            this.screengrabWindowActive = !this.screengrabWindowActive;
            this.photoWindowActive = false;
            this.filmWindowActive = false;
            this.reelWindowActive = false;
            this.settingsWindowActive = false;

            console.log(this.stagedItems[index].screenGrabs);

            this.focus = index;
        }
    }
}
</script>

<template>
  <div class="dash-container">
      <div class="navbar" v-bind:class="[ homeActive ? 'nav-solid' : 'nav-trans' ]">
          <div class="nav-left">
                <div class="menu-btn"></div>
                <div @click="toggleHome()" class="nav-logo">
                    <div class="logo"></div>
                </div>
          </div>
          <div class="nav-center">
              <ul>
                  <li @click="toggleHome()" class="nav-item" v-bind:class="[ homeActive ? 'nav-active' : 'nav-inactive' ]"><div class="nav-icon" id="nav-icon-home"></div><span>Dashboard</span></li>
                  <li @click="toggleFilm()" class="nav-item" v-bind:class="[ filmWindowActive || reelWindowActive ? 'nav-active' : 'nav-inactive' ]"><div class="nav-icon" id="nav-icon-film"></div><span>Add Film</span></li>
                  <li @click="togglePhoto()" class="nav-item" v-bind:class="[ photoWindowActive ? 'nav-active' : 'nav-inactive' ]"><div class="nav-icon" id="nav-icon-photo"></div><span>Add Photos</span></li>
              </ul>
          </div>
          <div class="nav-text">
              <ul>
                  <li @click="toggleSettings()" class="nav-item" id="nav-item-settings" v-bind:class="[ settingsWindowActive ? 'nav-active' : 'nav-inactive' ]"><div class="nav-icon" id="nav-icon-settings"></div>Profile</li>
                  <a class="live-site-button" href="https://tbrew1023.github.io/Nihal-Public" target="_blank"><li><span>View Live Site</span></li></a>
              </ul>
          </div>
      </div>

      <div id="film-window" v-bind:class="[ filmWindowActive ? 'window-section-active' : 'window-section-inactive' ]">
          <div class="inner-window-settings">
              <div class="film-upload-container">
                  
                  <div class="general-section film-upload-section">
                      <h2>General</h2>
                      <div class="film-inner">
                          <div class="inner-inputs">
                              <label>Project Title</label>
                              <input class="form-control" type="text" v-model="item.title" v-on:keyup.enter="() => { console.log('handling enter press'); }" />
                              <br>
                  
                              <label>Genre</label>
                              <input class="form-control" type="text" v-model="item.genre" v-on:keyup.enter="() => { console.log('handling enter press'); }" />
                              <br>
                  
                              <label>Summary</label>
                              <textarea style="resize: none" class="form-control" type="text" v-model="item.summary" v-on:keyup.enter="() => { console.log('handling enter press'); }"></textarea>
                          </div>
                      </div>
                  </div>

                  <div class="credits-section film-upload-section">
                      <h2>Credits</h2>
                      <div class="film-inner">
                          <ul>
                              <li v-for="(c, index) in item.credits" :key="index">
                                  <label class="left-label">Role</label>
                                  <input class="form-control" type="text" v-model="item.credits[index].role" v-on:keyup.enter="newCredit()">
                                  <label>Name</label>
                                  <input class="form-control" type="text" v-model="item.credits[index].name" v-on:keyup.enter="newCredit()">
                              </li>
                              <button class="btn" @click="newCredit()">Add Another Credit</button>
                          </ul>
                      </div>
                  </div>

                  <div class="upload-section screengrabs-section film-upload-section">
                      <h2>Upload Screengrabs (PNG, GIF, JPG, JPEG)</h2>
                      <div class="film-inner">
                          <ul>
                              <li v-for="(p, index) in photoCount" :key="index">
                                  <progress style="transition: 200ms" :value="screengrabUploadProgress" max="100" id="uploader" v-bind:class="[ uploadDone ? 'uploadDone' : '' ]">0%</progress>
                                  <input @change="handleScreengrabUpload" type="file" value="upload" class="fileButton fileButtonPhotos" :id="'fileButton' + focus" multiple/>
                              </li>
                              <button class="btn" @click="photoCount++">Add Another Screengrab</button>
                          </ul>
                      </div>
                  </div>

                  <div class="upload-section final-film-section film-upload-section">
                      <h2>Upload Final Film (MP4)</h2>
                      <div class="film-inner">
                          <ul>
                              <progress style="transition: 200ms" :value="uploadProgress" max="100" id="uploader" v-bind:class="[ uploadDone ? 'uploadDone' : '' ]">0%</progress>
                              <input @change="handleFilmUpload" type="file" value="upload" class="fileButton" id="fileButtonFilm" />
                          </ul>
                      </div>
                  </div>
              </div>
              <br>
              <div class="window-options">
                  <button class="btn btn-primary" @click="add()">Add to Portfolio</button>
                  <button class="btn btn-primary btn-cancel" @click="toggleHome()">Cancel</button>
              </div>
          </div>
      </div>

      <div id="reel-window" v-bind:class="[ reelWindowActive ? 'window-section-active' : 'window-section-inactive' ]">
          <div class="inner-window-settings">
              <div class="film-upload-container">
                  <div class="upload-section final-film-section film-upload-section">
                      <h2>Upload Reel (mp4)</h2>
                      <div class="film-inner">
                          <ul>
                              <progress style="transition: 200ms" :value="uploadProgress" max="100" id="uploader" v-bind:class="[ uploadDone ? 'uploadDone' : '' ]">0%</progress>
                              <input @change="handleReelUpload" type="file" value="upload" class="fileButton" id="fileButtonFilm" />
                          </ul>
                      </div>
                  </div>
              </div>
              <br>
              <div class="window-options">
                  <button class="btn btn-primary" @click="addReel()">Add to Portfolio</button>
                  <button class="btn btn-primary btn-cancel" @click="toggleHome()">Cancel</button>
              </div>
          </div>
      </div>

      <div id="photo-window" v-bind:class="[ photoWindowActive ? 'window-section-active' : 'window-section-inactive' ]">
          <div class="inner-window-settings">
              <div class="film-upload-container">
                  <div class="upload-section final-film-section film-upload-section">
                      <h2>Upload Photos (PNG, GIF, JPG, JPEG)</h2>
                      <div class="film-inner photo-inner">
                          <ul>
                              <li v-for="(p, index) in photoCount" :key="index">
                                  <progress style="transition: 200ms" :value="uploadProgress" max="100" id="uploader" v-bind:class="[ uploadDone ? 'uploadDone' : '' ]">0%</progress>
                                  <input @change="handlePhotoUpload" type="file" value="upload" class="fileButton" id="fileButtonPhotos" multiple/>
                              </li>
                              <button class="btn" @click="photoCount++">Add Another Photo</button>
                          </ul>
                      </div>
                  </div>
              </div>
              <br>
              <div class="window-options">
                  <button class="btn btn-primary" @click="toggleHome()">Done</button>
              </div>
          </div>
      </div>

      <div id="settings-window" v-bind:class="[ settingsWindowActive ? 'window-section-active' : 'window-section-inactive' ]">
          <div class="inner-window-settings settings-container">
              <div class="profile-upload" v-bind:style="{ backgroundImage: 'url(' + newBio.photo + ')' }">
                  <input @change="handleProfileUpload" type="file" value="upload" class="profileImgUpload" id="fileButtonFilm" />
                  <div class="photo-load-veil">
                      <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
                  </div>
                  <div class="replace-veil"><span>Replace</span></div>
              </div>
              <label>Bio</label>
              <br>
              <textarea v-model="newBio.text"></textarea>
              <br>
              <div class="window-options">
                  <button class="btn btn-primary" @click="addBio()">Update Profile</button>
                  <button class="btn btn-primary btn-cancel" @click="toggleHome()">Cancel</button>
              </div>
          </div>
      </div>

      <div id="screengrab-window" v-bind:class="[ screengrabWindowActive ? 'window-section-active' : 'window-section-inactive' ]">
          <div class="inner-window-settings">
              <div class="film-upload-container">
                  <div class="upload-section final-film-section film-upload-section">
                      <h2>Upload Screengrabs (PNG, GIF, JPG, JPEG)</h2>
                      <div class="film-inner photo-inner">
                          <ul>
                              <li v-for="(p, index) in photoCount" :key="index">
                                  <progress style="transition: 200ms" :value="uploadProgress" max="100" id="uploader" v-bind:class="[ uploadDone ? 'uploadDone' : '' ]">0%</progress>
                                  <input @change="handleNewScreenGrab" type="file" value="upload" class="fileButton fileButtonPhotos" :id="'fileButton' + focus" multiple/>
                              </li>
                              <button class="btn" @click="photoCount++">Add Another Screengrab</button>
                          </ul>
                      </div>
                  </div>
              </div>
              <br>
              <div class="window-options">
                  <button class="btn btn-primary" @click="toggleHome()">Done</button>
              </div>
          </div>
      </div>

      <div class="content" v-bind:class="[ filmWindowActive || settingsWindowActive || photoWindowActive || reelWindowActive || screengrabWindowActive ? 'content-blur-active' : 'content-blur-inactive' ]">
          <div class="section-films">
                  <h2>Films</h2>
                  <ul class="list-group reel-container">
                      <li class="list-group-item project-item" id="vid-reel">
                          <button style="float: right" @click="toggleReel()" class="badge reel-badge">Replace</button>
                          <div class="vid-section">
                              <video class="reel-video" controls muted autoplay>
                                  <source :src="filmReel" type="video/mp4" />
                              </video>
                              <div class="vid-text">
                                  <h1>Film Reel</h1>
                                  <strong><p style="opacity:0.5">Compilation</p></strong>
                                  <p class="summary">Shown on the landing page of the site.</p>
                              </div>
                          </div>
                      </li>
                  </ul>
                  <ul class="list-group">
                      <li class="list-group-item project-item" v-for="(i, index) in stagedItems" :id="'vid' + index" :key="index">

                          <button style="float: right" @click="removeProject(i)" class="badge">Remove</button>
                          <button style="float: right" :id="'update' + index" @click="handleUpdate(index, i)" class="badge" v-bind:class="( updateActive ? 'reel-badge update-active' : 'update-inactive' )">Save Changes</button>
                          <div class="vid-section">
                              <div class="video-container">
                                  <video v-if="i.finalFilm" controls muted autoplay>
                                      <source :src="i.finalFilm  + '#t=' + i.startTime" type="video/mp4" />
                                  </video>
                                  <div v-if="!i.finalFilm" class="empty-video">
                                      <div class="empty-video-inner">
                                          <div class="empty-video-icon"></div>
                                          <p>No Film Uploaded</p>
                                      </div>    
                                  </div>
                              </div>
                              <div class="vid-text">
                                  <!--title-->
                                  <div class="update-form-container">
                                      <input style="font-weight: bold" type="text" class="edit-title edit" v-model="stagedItems[index].title" @input="stageUpdate(index, i)" />
                                      <div class="edit-icon"></div>
                                  </div>
                                  <!--genre-->
                                  <div class="update-form-container">
                                      <input type="text" class="edit-genre edit" v-model="stagedItems[index].genre" @input="stageUpdate(index, i)" />
                                      <div class="edit-icon"></div>
                                  </div>
                                  <!--summary-->
                                  <div class="update-form-container">
                                      <textarea v-model="stagedItems[index].summary" class="summary edit-summary edit" @input="stageUpdate(index, i)"></textarea>
                                      <div class="edit-icon"></div>
                                  </div>
                              </div>
                          </div>
                          
                          <br>
                          <!--start time-->
                          <div v-if="i.finalFilm" class="start-time-container">
                              <label style="font-size: 14px">Start Time (Seconds)</label>
                              <input type="number" class="edit-start-time" v-model="stagedItems[index].startTime" @input="stageUpdate(index, i)" />   
                          </div>
                          <br>
                          <!--screengrabs-->
                          <ul class="images">
                              <li v-for="(p, photoIndex) in stagedItems[index].screenGrabs" :key="photoIndex" class="screengrab-container">
                                  <div @click="handleRemoveScreengrab(index, photoIndex)" class="screengrab-delete-veil"><span>Remove</span></div>
                                  <img :src="p" />
                              </li>
                              <li class="screengrabs-button-container">
                                  <div @click="toggleScreengrab(index)" class="screengrabs-button"><span style="opacity:1"><strong>+</strong><br>Add Screengrabs</span></div>
                              </li>
                          </ul>
                          <br>
                          <!--credits-->
                          <div class="credits-container">
                              <p class="credit" style="display: inline-flex" v-for="(c, index) in i.credits" :key="index"><span>{{c.role + " "}}</span>&nbsp;<strong>{{c.name + " "}}</strong>&nbsp;</p>
                          </div>
                      </li>
                  </ul>
          </div>
          
          <div class="section-photos">
              <h2 class="photo-header">Photos</h2>
              <div class="photos-container">

                  <div v-if="showPhotos" class="grid-container">
                      <div class="photo" @click="removePhoto(p)" v-for="(p, index) in photos" :key="index">
                          <img :src="p.thumbSrc100" alt="" width="150" height="150"/>
                          <div class="photo-delete"><span>Remove</span></div>
                          <div class="photo-load-veil-small">
                              <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
                          </div>
                      </div>
                  </div>

                  <h3 style="margin-left: 24px;font-weight: normal; font-size: 14px; opacity: 0.3; color: white" v-if="photos.length == 0 || !showPhotos">No photos uploaded.</h3>

              </div>
          </div>
      </div>
  
  </div>
</template>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
$colorText: #555;
$colorMain: #212225;
$colorDark: #131417;
$colorLight: rgba(white, 0.3);
$colorLighter: #43454a;
$colorLink: #ff7065;
$colorWhite: #eee;
$colorGreen: #66BB6A;
$colorYellow: #FBC02D;

body {
    opacity: 0;
    margin: 0px;
    font-family: 'Yantramanav', sans-serif;
    background: $colorMain;
    //background: blue;
    color: white;
    //overflow: hidden;

    p {
        font-size: 16px;
    }

    .credits-container {
        margin-top: 14px !important;
    }

    .credit {
        font-size: 14px;

    }
}

.menu-btn {
    background:blue;
    margin-left: 24px;
    width: 30px;
    height: 30px;
}

.auth {
    position: fixed;
    background: gray;
    height: 100vh;
    width: 100%;
    z-index: 9999;
    overflow: hidden;
}

.live-site-button {
    background: transparent;
    //color: white;
    text-decoration: none;
    font-size: 14px;
    //margin-right: -12px;
    //border-radius: 0px 0px 0px 24px;

    span {
        line-height: normal;
        background: transparent;
        border: $colorGreen 2px solid;
        transition: 200ms;
        padding: 3px 6px 3px 6px;
        color: $colorGreen;
        font-weight: bold;
        border-radius: 6px;
    }

    &:hover {
        span {
            background: $colorGreen;
            color: white;
        }
    }

}

.nav-solid {
    background: #212225;
}

.nav-trans {
    background: transparent;
    box-shadow: none !important;
}

.navbar {
    position:fixed;
    z-index: 999;
    display: flex;
    align-items: center;
    //justify-content: center;
    width: 100%;
    height: 60px;
    box-shadow: 0px 5px 24px -6px rgba(0,0,0,0.4);
    transition: 300ms;

    h3 {
        font-weight: normal;
        font-size: 20px;
    }

    li {
        //opacity: 0.3;
        height: 60px;
        line-height: 60px;
        cursor: pointer;
        transition: 200ms;
    }

    .nav-item {
        //opacity: 0.4;
        font-size: 14px;
        font-weight: bold;
    }


}

.settings-button-container {
    //background: red;
    display: flex;
    justify-content: center;
    align-items: center;
}

.settings-button {
    //background:rgba(white, 0.2);
    border: solid white 2px;
    opacity: 1 !important;
    //border-radius: 6px;
    height: 25px;
    line-height: 25px;
    padding-left: 6px;
    padding-right: 6px;
    transition: 200ms;
    //font-weight: bold;
    //font-size: 14px;
}

#nav-item-settings {
    &:hover {
        .nav-icon {
            transform: rotate(135deg);
        }
    }
}

.nav-active {
    opacity: 1 !important;
}

.nav-inactive {
    opacity: 0.3;

    &:hover {
        opacity: 0.6;
    }
}

.nav-left {
    display: flex;
}

.logo {
    background-position: center;
    background-size: contain;
    height: 30px;
    width: 30px;
    background-image: url('../assets/logo.svg');
    background-repeat: no-repeat;
    margin-left: 18px;
}

.nav-logo {
    position: absolute;
    left: 0px;
    margin-left: 24px;
    cursor: pointer;

    span {
        opacity: 0.6;
    }
}

.nav-text {
    position: absolute;
    right: 0px;
    margin-right: 12px;
    display: flex;
    justify-content: center;
    align-items: center;

    ul {
        //background: red;
        display: inline-flex;
        text-align: center;

        li {
            margin-left: 12px;
            margin-right: 12px;
            display: flex;
            justify-content: center;
            align-items: center;
        }
    }
}

.nav-center {
    position: absolute;
    right: 0px;
    left: 0px;
    margin: auto;
    width: 50%;
    display: flex;
    justify-content: center;
    align-items: center;

    ul {
        //background: red;
        display: inline-flex;
        text-align: center;
        list-style: none;

        li {
            margin-left: 12px;
            margin-right: 12px;
            display: inline-flex;
            justify-content: center;
            align-items: center;
        }
    }
}

.nav-icon {
    //background: red;
    filter: invert(1);
    //opacity: 0.3;
    width: 24px;
    height: 24px;
    margin-right: 12px;
    background-position: center;
    background-repeat: no-repeat;
    background-size: contain;
    transition: 300ms;
}

#nav-icon-home {
    background-image: url('../assets/home-24px.svg');
}

#nav-icon-film {
    background-image: url('../assets/videocam-24px.svg');
}

#nav-icon-photo {
    background-image: url('../assets/camera_alt-24px.svg');
}

#nav-icon-settings {
    background-image: url('../assets/settings-24px.svg');
}

.dash-container {
    width: 100%;
    
    ul {
        list-style: none;
        padding: 0px;
    }
}

.content {
    display: inline-flex;
    margin-top: 60px;
    width: 100%;
}

.section {
    width: 100%;
}

.film-upload-container {
    //position: fixed;
    //padding: 30px;
    //background: $colorLight;
    border-radius: 12px;
    //width: 340px;
    //margin-top: 60px;
    width: 100%;
    //height: 700px;
    //padding-bottom: 36px;

    display: inline-flex;

    h2 {
        margin: 0px;
        padding: 12px 6px 6px 6px;
    }

    .film-upload-section {
        width: 300px;
        margin: 0px;
        border-radius: 0px;
        padding: 0px;
        margin: 24px;
        border-radius: 6px;

        .film-inner {
            padding: 30px;
            display: flex;
            justify-content: center;
            align-items: center;
            height: calc(100% - 96px);
    
            ul {
                margin: 0px;
                padding: 0px;
            }

            textarea {
                width: 95%;
                max-width: 260px;
                max-height: 200px;
                height: 80px;
                margin-top: 6px;
                background: rgba(white, 0.1);
                border: none;
                color: white;
                padding: 3px 6px 3px 6px;
            }
        }

         h2 {
             background:rgba(white,0.1);
             font-size: 12px;
             padding: 6px 12px 6px 12px !important;
         }
    }

    ul {
        //padding: 30px;
        //display: flex;
        //justify-content: center;
    }

    .general-section {
        background: #131417;
        border-radius: 6px 0px 0px 6px;
        padding: 0px;
        width: 300px;
    }

    .credits-section {
        background:#131417;
        width: 360px;
    }

    .upload-section {
        background:#131417;
        width: 300px;
    }

    .final-film-section {
        background: #131417;
    }
    
}


.photo-upload-inner {
    h2 {
        color: white;
        margin-left: 24px !important;
        
    }
}

$imageDims: 100px;

.photos-container {
    //margin-top: 120px;
    background: $colorDark;
    color: $colorWhite;
    margin-top: 20px;
    //padding: 0px;
    border-radius: 6px;
    //height: 700px;
    min-width: 360px;
    padding: 30px;

    .grid-container {
        //background: blue;
        width: 100%;
        height: max-content;
        display: grid;
        grid-template-columns: 1fr 1fr 1fr;
        grid-gap: 18px;

        .photo-delete {
            opacity: 0;
            transition: 100ms;
            background: 100%;
            height: 100%;
            margin: 0px;
            background: rgba(black, 0.8);
            text-align: center;
            font-weight:bold;
            line-height: $imageDims;
            color: $colorLink;
            z-index:6;

            span {
                transition: 150ms;
                //transform: translateY(-64px);
            }
        }

        .photo {
            background:rgba(0,0,0,0.6);
            /* PUT LOADER HERE */
            width: $imageDims;
            height: $imageDims;
            border-radius: 6px;
            background-repeat: no-repeat;
            background-size: cover;
            background-position: center;
            cursor: pointer;
            z-index: 5;

            img {
                position: absolute;
                z-index: -1;
                object-fit: cover;
                border-radius: 6px;
                height: $imageDims;
                width: $imageDims;
            }

            &:hover {
                .photo-delete {
                    opacity: 1 !important;
                    z-index: 10;
                    
                    span {
                        transition: 150ms;
                        //transform: translateY(0px);
                    }
                }
            }
        }
    }
}

.photo-header {
    //cursor: pointer;
}

.section-photos {
    margin: 36px 30px 30px 30px;
    padding-top: 12px;
}

.section-films {
    //background: $colorMain;
    width: 100%;
    padding-top: 48px;
    //padding: 30px;
    //margin-left: 560px;

    h2 {
        margin-left: 32px;
    }
}

.content-blur-active {
    transition: 200ms;
    filter: blur(24px) brightness(0.7);
}

.content-blur-inactive {
    transition: 200ms;
    filter: blur(0px);
}

.window-section-active {
    //background: black;
    width: 100%;
    height: 100vh;
    opacity: 1;
    position: fixed;
    margin: auto;
    left: 0px;
    right: 0px;
    top: 0px;
    bottom: 0px;
    z-index: 50;
    border-radius: 6px;
    //padding: 48px;
    transition: 300ms;
    overflow: hidden;

    .inner-window-settings {
        background: rgba(white, 0.05);
        //max-width: 90%;
        width: fit-content;
        height: fit-content;
        //max-height: 800px;
        //min-height: 500px;
        position: absolute;
        margin: auto;
        top: 0px;
        right: 0px;
        bottom: 0px;
        left: 0px;
        transition: 200ms;
        border-radius: 6px;
        padding:12px 12px 24px 12px;
        //display:flex;
        //padding-bottom: 70px;
    }
}

.submit-area {
    width: 300px;
    background: $colorText;
    //display: flex;
    //justify-content: center;
    //align-items: center;

    h2 {
        font-size: 12px;
        padding: 6px 12px 6px 12px;
        margin: 0px;
        background: rgba(white,0.2);
    }
}

.window-section-inactive {
    //background: black;
    opacity: 0;
    width: 100%;
    height: 100vh;
    position: fixed;
    margin: auto;
    left: 0px;
    right: 0px;
    top: 0px;
    bottom: 0px;
    z-index: 50;
    border-radius: 6px;
    //padding: 48px;
    transition: 300ms;
    pointer-events: none;

    .inner-window-settings {
        transform: scale(0.9);
        background: #212225;
        //max-width: 90%;
        width: fit-content;
        height: fit-content;
        //max-height: 800px;
        //min-height: 500px;
        position: absolute;
        margin: auto;
        top: 0px;
        right: 0px;
        bottom: 0px;
        left: 0px;
        transition: 200ms;
        border-radius: 6px;
        padding:12px;
        //display:flex;
    }
}

#film-window {
    .inner-window {
        //width: max-content;
        //background: $colorText;
    }
}

#photo-window {
    .inner-window {
        //width: max-content;
        background: $colorYellow;
    }
}

.settings-container {
    display: block !important;
    padding: 48px !important;
}

.photo-load-veil {
    position: absolute;
    background: transparent;
    pointer-events: none;
    width: 150px;
    height:150px;
    margin-top:-150px;
    z-index: -2;
    display: flex;
    justify-content: center;
    align-items: center;
}

.photo-load-veil-small {
    position: absolute;
    background: transparent;
    pointer-events: none;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100px !important;
    height: 100px !important;
    margin-top: -100px;
    z-index: -2;
    opacity: 0.2;
    //transform: scale(0.8);
}

// ---------- loader for photos ----------

.lds-ring {
    display: inline-block;
    //position: absolute;
    width: 64px !important;
    height: 64px !important;
    margin: 0px;
    transform: scale(0.8);
  }
  .lds-ring div {
    box-sizing: border-box;
    display: block;
    position: absolute;
    width: 50px !important;
    height: 50px !important;
    margin: 8px;
    border: 4px solid #fff;
    border-radius: 50%;
    animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
    border-color: #fff transparent transparent transparent;
  }
  .lds-ring div:nth-child(1) {
    animation-delay: -0.45s;
  }
  .lds-ring div:nth-child(2) {
    animation-delay: -0.3s;
  }
  .lds-ring div:nth-child(3) {
    animation-delay: -0.15s;
  }
  @keyframes lds-ring {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

  //---------- end of photo loaders ----------

.profile-upload {
    background: $colorLight;
    width: 150px;
    height: 150px;
    margin-left: auto;
    margin-right: auto;
    margin-bottom: 36px;
    border-radius: 6px;
    //transition: 150ms;
    background-position: center;
    background-size:cover;
    background-repeat: no-repeat;
    cursor: pointer;
    &:hover {
        .replace-veil {
            opacity: 1;
        }
    }
}

#settings-window {
    .inner-window {
        //width: max-content;
        background: $colorGreen;
    }

    input[type="text"] {
        background: rgba(255, 255, 255, 0.1);
        border: none;
        width: 400px;
        color: white;
    }

    textarea {
        background:rgba(255, 255, 255, 0.1);
        border: none;
        border-radius: 6px;
        width: 400px;
        height: 200px;
        padding:12px;
        margin-bottom: 36px;
        color: white;
    }
}

.replace-veil {
    background: rgba(0,0,0,0.8);
    color: white;
    line-height: 150px;
    text-align: center;
    opacity: 0;
    transition: 100ms;
    pointer-events: none;
    margin-top: -150px;
    //border-radius: 6px;

    span {
        font-weight: bold;
    }
}

#vid-reel {
    margin-top: 0px;
    background:rgba(white,0.04);
    padding-bottom: 24px;
    //border: $colorLight solid 2px;

    img {
        margin-bottom: 0px;
    }

    .badge {
        background: transparent;
        border-color: white;
        color: white;

        &:hover {
            background: white;
            color: black;
        }
    }
}

#vid0 {
    //margin-top: 20px;
}

input[type="text"] {
    background:rgba(white, 0.1);
    border: none;
    padding: 3px 6px 3px 6px;
    //border-radius: 6px;
    margin-bottom: 6px;
    margin-top: 6px;
    color: white;
    //transition: 200ms;
    width: 90px;
}

//---------- project update stuff ----------

$iconDims: 24px;

.input-container {
    //background: red;
    display: flex;
    //line-height: 30px;
}

.update-form-container {
    display: flex;
    border-radius: 6px;
    //transition: 100ms;
    margin-bottom: 12px;
    border: 2px transparent solid;
    //width: calc(100% + 28px) !important;
    //align-items: center;
    //background:blue;

    &:hover {
        //background: $colorMain !important;
        border: 2px white solid;
        .edit-icon {
            width: $iconDims;
            height: $iconDims;
            //margin-left: 12px;
            opacity: 1;
            transform: scale(1) translate(-12px, -10px);
        }
    }

    .edit:focus {
        background: $colorMain !important;
        color: white !important;
        //border: 2px white solid;

    }
}

.edit {
    border-radius: 0px 3px 3px 3px;
    background: transparent !important;
    margin: 0px !important;
    //transition: 100ms;
    width: 100% !important;
    padding: 6px 12px 6px 12px !important;
}

.edit-icon {
    background: white;
    border-radius: 30px;
    width: $iconDims;
    height: $iconDims;
    //transition:100ms;
    opacity: 0;
    //transform: translateX(24px);
    margin-right: -$iconDims;
    background-image: url("../assets/create-24px.svg");
    background-size: 80%;
    background-position: center;
    background-repeat: no-repeat;
    //transform: scale(0.5) translate(-20px, -18px);
}

.edit-title {
    font-size: 32px;
    padding-left: 12px;
}

.edit-genre {
    font-size: 16px;
    color: rgba(#fff, 0.5) !important;
}

.edit-summary {
    font-size: 16px;
    color: white;
    font-family: inherit;
    height: 100px;
    margin-top: 6px;
    resize: none;
    line-height: 1.5;
    border: none;
    padding-right: 32px !important;
}

//---------- UPDATE STUFF ----------

.update-inactive {
    opacity: 0;
    pointer-events: none;
    margin-right: 18px;
    border-color: $colorGreen !important;
    color: $colorGreen !important;
    //transform: transformY(12px);

    &:hover {
        background: $colorGreen !important;
        color: white !important;
    }
}

.update-active {
    opacity: 1;
    margin-right: 18px;
    border-color: $colorGreen !important;
    color: $colorGreen !important;

    &:hover {
        background: $colorGreen !important;
        color: white !important;
    }
}

//---------- GENERAL JAZZ ----------

.general-section, .credits-section, .final-film-section, .screengrabs-section {
    background: rgba(white, 0.1) !important;
    border-radius: 6px;
    margin: 12px 0px 12px 0px;
    padding-left: 24px;
    padding-right: 12px;
}

.screengrabs-button-container {
    //background: blue;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-left: 0px;
    //margin-top: -12px;
}

.screengrabs-button {
    background: $colorMain;
    //height: 24px;
    width: 160px;
    height: 88px;
    border-radius: 6px;
    //border: 2px white solid;
    text-align: center;
    display: flex;
    justify-content: center;
    align-items: center;
    color: white;
    font-size: 12px;
    cursor: pointer;
    transition: 100ms;
    margin: 0px 18px 16px 0px;

    strong {
        font-size: 18px;
        color: white;
    }

    &:hover {
        background: $colorLighter;
    }
}

.screengrab-container {
    cursor: pointer;

    img {
        z-index: -1 !important;
    }
}

.screengrab-delete-veil {
    opacity: 0;
    //transition: 100ms;
    background: 100%;
    height: 90px;
    width: 160px;
    margin: 0px;
    background: rgba(black, 0.8);
    text-align: center;
    font-weight:bold;
    line-height: 90px;
    color: $colorLink;
    z-index:9999 !important;
    border-radius: 6px;
    //margin-top: 88px !important;
    position: absolute;

    span {
        //transition: 150ms;
        opacity: 1 !important;
        //transform: translateY(-64px);
    }

    &:hover {
        opacity: 1 !important;
        //background: blue !important;
    }
}

.credits-section {
    //padding: 0px 24px 0px;

    ul {
        margin-bottom: 0px;
        padding-top: 18px;
    }
}

.general-section {
    padding: 24px;

    input {
        width: 95%;
    }
}

.profileImgUpload {
    //background:red;
    width: 150px;
    height: 150px;  
    opacity: 0;
    cursor: pointer;
    border-radius: 6px;
    
    &:hover {
       //background: blue;
    }
}

/*.profileImgUpload::before {
    content: 'o boy';
    display:none;
}

.profileImgUpload::after {
    content: '';
}

.profileImgUpload::-webkit-file-upload-button {
    visibility: hidden;
}*/

.fileButton::-webkit-file-upload-button {
    visibility: hidden;
}

.fileButton::before {
    content: 'Choose file';
    background: transparent;
    border: 2px solid white;
    padding: 3px 6px 3px 6px;
    font-size: 12px;
    //margin-right: 24px;
    transition: 200ms;
    cursor: pointer;
}

.fileButton::after {
    //content: 'no file chosen';
    //background: red;
}

.fileButton {
    font-size: 12px;
    cursor: pointer;
    margin-top: 6px;

    &:hover {
        &::before {
            color: $colorDark;
            background: white;
        }
    }
}

.final-film-section {

}

label {
    margin-right: 6px;
    margin-left: 6px;
    padding-bottom: 6px;
    font-size: 12px;
}

button {
    background: transparent;
    color: $colorGreen;
    border: 2px $colorGreen solid;
    padding: 3px 6px 3px 6px;
    //border-radius: 6px;
    cursor: pointer;
    transition: 150ms;
    margin-top: 18px;
    font-size: 12px;

    &:hover {
        color: white;
        background: $colorGreen;
    }
}

.badge {
    margin-top: 0px;
    background: $colorDark;
    border: 2px $colorLink solid;
    color: $colorLink;
    font-weight: bold;
    border-radius: 6px;
    font-size: 14px !important;

    &:hover {
        color: white;
        background: $colorLink;
    }
}

h2 {
    //margin-bottom: 18px;
    color: white;
    //opacity: 0.5;
    font-size: 14px;
    font-weight: normal;
    margin-bottom: 0px;
    margin-left: 0px;
}

.upload-section {
    //padding-top: 6px;
    padding-bottom: 0px;
    padding-right: 24px;
    margin-bottom: 0px;
    //height: 60px;

    input {
        display: block;
    }

    ul {
        margin-bottom: 0px;
    }
}

.window-options {
    //position: absolute;
    display: inline-flex;
    //background: red;
    justify-content: center;
    align-items: center;
    padding: 24px 0px 24px 0px;
    width: 100%;
}

.btn-primary {
    //width: 150px;
    background: $colorGreen;
    border-radius: 6px;
    color: white;
    height: 36px;
    //color: $colorGreen;
    border: solid 2px $colorGreen;
    //border-radius: 6px;
    font-size: 14px;
    font-weight: bold;
    margin: 0px 12px 0px 0px;

    &:hover {
        filter: brightness(0.6);
    }
}

.btn-cancel {
    //position: absolute;
    background: transparent;
    color: white;
    border-color: transparent;
    //width: 150px;

    &:hover {
        background: transparent;
    }
}

input[type="file"] {
    //font-size: 16px;
    //padding-top: 20px;
}

.fileButton:focus {
    
}

.fileButtonPhotos::before {
    //background: red;

    content: 'Choose files';
}

.left-label {
    margin-left: 0px;
}

progress {
    //appearance: none;
    width: 100%;
    height: 12px;
    margin-bottom: 12px;
    margin-top: 24px;
    //border-radius: 24px;
}

.uploadDone {
    background: green;
}

progress[value] {
    //background-color: $colorYellow;
}

/*.general-section {
    //background: $colorDark;

    padding: 12px;

    label {
        margin-bottom: 12px;
    }

    input {
        width: 92%;
        margin-top: 6px;
    }
}*/

.reel-container {
    //background: black;
    margin-top: 18px;

    ul {
        list-style: none !important;
    }

    video {
        margin-bottom: 0px !important;
    }
}

.empty-video {
    background: rgba(black, 0.6);
    height: 225px;
    width: 400px;
    border-radius: 6px;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    margin-right: 30px;

    .empty-video-inner {
        opacity: 0.6;

        .empty-video-icon {
            //background: blue;
            height: 45px;
            width: 45px;
            margin-left: auto;
            margin-right: auto;
            margin-bottom: 24px;
            background-image: url('../assets/videocam_off-24px.svg');
            background-position: center;
            background-size: contain;
            background-repeat: no-repeat;
            filter: invert(1);
        }
    }
}

.project-item {
    background: $colorDark;
    padding: 30px;
    margin: 30px;
    //margin-left: 0px;
    margin-right: 0px;
    border-radius: 6px;

    input[type="number"] {
        text-align: center;
        margin-bottom: 12px;
        width: 32px;
        background: rgba(white, 0.1);
        color: white;
        border:none;
        padding: 3px 6px 3px 6px;
        border-radius: 6px;
    }

    input[type="number"]::-webkit-inner-spin-button {
        -webkit-appearance: none;
    }

    p {
        margin-top: 8px;
    }

    .item-header {
        background: $colorMain;
        height: 24px;
        width: 100%;
    }

    h1 {
        margin-bottom: 0px;
    }

    .vid-section {
        display: inline-flex;

        .vid-text {
            max-width: 640px;
            min-width: 500px;
        }
    }

    video {
        border-radius: 6px;
        margin-right: 30px;
        margin-bottom: 18px;
        width: 400px;
        height: 225px;
    }

    //screengrabs

    img {
        margin: 0px 18px 12px 0px;
        border-radius: 6px;
        width: 160px;
    }

    .images {
        margin-left: 3px;
        display: inline-flex;
        list-style: none;
        padding: 0px;
        flex-wrap: wrap;
        max-width: 1250px;
    
        li:nth-child(1) {
            width: 0px !important;
            //display: none;
        }
    }

    span {
        opacity: 0.6;
    }
}

.summary {
    //max-width: 512px;
    font-weight: normal;
}

.veil {
    position: fixed;
    background:#212225;
    height: 100vh;
    width: 100%;
    z-index: 25;

    .login {
        background:#131417;
        z-index: 30;
        border-radius: 6px;
        width: 300px;
        height: 300px;
        position: fixed;
        margin: auto;
        left: 0px;
        right: 0px;
        top: 0px;
        bottom: 0px;
        padding: 24px;
    
        input::before {
    
        }
    }
}

// ---------- smaller devices ----------

@media only screen and (max-width: 1000px) {
    .images {
        display: block;
    }
}
</style>
