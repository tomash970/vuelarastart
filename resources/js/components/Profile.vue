<style type="text/css">
    .widget-user-header{
        background-position: center center;
        background-size: cover;
        height: 300px;
    }
</style>
<template>
    <div class="container">
        <div class="row ">
            <div class="col-md-12 mt-3">
                <!-- Widget: user widget style 1 -->
                <div class="card card-widget widget-user">
                  <!-- Add the bg color to the header using any of the bg-* classes -->
                  <div class="widget-user-header text-white" style="background-image: url('./img/user-cover.jpg')">
                    <h3 class="widget-user-username">{{this.form.name}}</h3>
                    <h5 class="widget-user-desc" style="text-transform: uppercase">{{this.form.type}}</h5>
                  </div>
                  <div class="widget-user-image">
                    <img class="img-circle" :src="getProfilePhoto()" alt="User Avatar">
                  </div>
                  <div class="card-footer">
                    <div class="row">
                      <div class="col-sm-4 border-right">
                        <div class="description-block">
                          <h5 class="description-header">3,200</h5>
                          <span class="description-text">SALES</span>
                        </div>
                        <!-- /.description-block -->
                      </div>
                      <!-- /.col -->
                      <div class="col-sm-4 border-right">
                        <div class="description-block">
                          <h5 class="description-header">13,000</h5>
                          <span class="description-text">FOLLOWERS</span>
                        </div>
                        <!-- /.description-block -->
                      </div>
                      <!-- /.col -->
                      <div class="col-sm-4">
                        <div class="description-block">
                          <h5 class="description-header">35</h5>
                          <span class="description-text">PRODUCTS</span>
                        </div>
                        <!-- /.description-block -->
                      </div>
                      <!-- /.col -->
                    </div>
                    <!-- /.row -->
                  </div>
                </div>
                <!-- /.widget-user -->
            </div>
        </div>
        <div class="row">
            <div class="col-md-12">
                
                <div class="card">
                  <div class="card-header p-2">
                    <ul class="nav nav-pills">
                      <li class="nav-item"><a class="nav-link active show" href="#activity" data-toggle="tab">Activity</a></li>
                      <li class="nav-item"><a class="nav-link" href="#settings" data-toggle="tab">Settings</a></li>
                    </ul>
                  </div><!-- /.card-header -->
                  <div class="card-body">
                    <div class="tab-content">
                      <div class="tab-pane active show" id="activity">
                        <h2>Display user activity</h2>
                      </div>

                      <div class="tab-pane" id="settings">
                        <form class="form-horizontal">
                          <div class="form-group">
                            <label for="inputName" class="col-sm-2 control-label">Name</label>

                            <div class="col-sm-10">
                              <input type="text" v-model="form.name" class="form-control" id="inputName" placeholder="Name" :class="{'is-invalid': form.errors.has('name')}">
                              <has-error :form="form" field="name"></has-error>
                            </div>
                          </div>
                          <div class="form-group">
                            <label for="inputEmail" class="col-sm-2 control-label">Email</label>

                            <div class="col-sm-10">
                              <input type="email" v-model="form.email" class="form-control" id="inputEmail" placeholder="Email" :class="{'is-invalid': form.errors.has('email')}">
                              <has-error :form="form" field="email"></has-error>
                            </div>
                          </div>
                          <div class="form-group">
                            <label for="inputExperience" class="col-sm-2 control-label">Experience</label>

                            <div class="col-sm-10">
                              <textarea class="form-control" v-model="form.bio" id="inputExperience" placeholder="Experience" :class="{'is-invalid': form.errors.has('bio')}"> </textarea>
                              <has-error :form="form" field="bio"></has-error>
                            </div>
                          </div>
                          <div class="form-group">
                            <label for="inputSkills" class="col-sm-2 control-label">PIcture o you</label>

                            <div class="col-sm-10">
                              <input type="file" @change="updateProfile" class="form-control-file"  placeholder="Skills">
                            </div>
                          </div>

                          <div class="form-group">
                            <label for="inputName" class="col-sm-10 control-label">Passport(leave empty if not changing)</label>

                            <div class="col-sm-10">
                              <input type="password" class="form-control" v-model="form.password" id="inputName" placeholder="Passport(leave empty if not changing)" :class="{'is-invalid': form.errors.has('password')}">
                              <has-error :form="form" field="password"></has-error>
                            </div>
                          </div>

                          <div class="form-group">
                            <div class="col-sm-offset-2 col-sm-10">
                              <button @click.prevent="updateInfo" type="submit" class="btn btn-info">Update</button>
                            </div>
                          </div>
                        </form>
                      </div>
                      <!-- /.tab-pane -->
                    </div>
                    <!-- /.tab-content -->
                  </div><!-- /.card-body -->
                </div>
                <!-- /.nav-tabs-custom -->
          
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data(){
            return {
                form: new Form({
                    id:'',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                  })
            }
        }, 
        mounted() {
            console.log('Component mounted.')
        },
        methods:{
            getProfilePhoto() {
                let photo = (this.form.photo.length >100) ? this.form.photo : "img/profile/" + this.form.photo;
                return photo;
                //return "img/profile/" + this.form.photo;
            },
            updateInfo() {
                this.$Progress.start();

                this.form.put('api/profile')
                .then(() => {
                    Fire.$emit('AfterCreate');
                    this.$Progress.finish();
                })
                .catch(() => {
                    this.$Progress.fail();
                })
            },
            updateProfile(e) {
                let file = e.target.files[0];
                console.log(file);
                let reader = new FileReader();
                // reader.onloadend = function(){
                //     console.log('RESULT', reader.result);
                // }

                if(file['size'] < 2111775) {
                    reader.onloadend = (file) => {
                        this.form.photo = reader.result;
                    }
                    reader.readAsDataURL(file);
                }else{
                    swal.fire({
                        type: 'error',
                        title:'Ooops...',
                        text: 'You are trying to upload to large file',
                    });
                }
                
            }
        },
        created(){
            axios.get("api/profile")
            .then(({data}) => (this.form.fill(data)))
        }
    }
</script>