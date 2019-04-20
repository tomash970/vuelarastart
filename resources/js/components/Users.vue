<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminOrAuthor()">
            
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                    <button type="button" class="btn btn-primary" @click="newModal">Add New <i class="fas fa-user-plus fa-fw"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody>
                  <tr>
                    <th>ID</th>
                    <th>Photo</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Biography</th>
                    <th>Created at</th>
                    <th>Modify</th>
                  </tr>
                  <tr v-for="user in users.data" :key="user.id">
                    <td>{{user.id}}</td>
                    <td>{{user.photo}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td>{{user.type | upText}}</td>
                    <td>{{user.bio}}</td>
                    <td>{{user.created_at | myDate}}</td>
                    <td>
                        <a @click="editModal(user)">
                            <i class="fa fa-edit text-blue"></i>
                        </a>
                        /
                        <a @click="deleteUser(user.id)">
                            <i class="fa fa-trash text-red"></i>
                        </a>
                    </td>
                  </tr>
                  
                    </tbody>
                </table>
              </div>
              <!-- /.card-body -->
              <div class="card-footer">
                  <pagination :data="users" @pagination-change-page="getResults"></pagination>

              </div>
              <!-- /.card-footer -->
            </div>
            <!-- /.card -->
          </div>
        
        </div>
        <div v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
          <div class="modal-dialog modal-dialog-cetered" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="addNewLabel">{{editmode ? 'Update User' : 'Add New'}}</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
              <form @submit.prevent="editmode ? updateUser() : submitUser()">
                  <div class="modal-body">
                        <div class="form-group">
                          <input v-model="form.name" type="text" name="name" placeholder="Name" 
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                          <has-error :form="form" field="name"></has-error>
                        </div>

                        <div class="form-group">
                          <input v-model="form.email" type="text" name="email" placeholder="email" 
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                          <has-error :form="form" field="email"></has-error>
                        </div>

                        <div class="form-group">
                          <textarea v-model="form.bio" name="bio" placeholder="Short bio dor user(Optional)" 
                            class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                          <has-error :form="form" field="bio"></has-error>
                        </div>

                        <div class="form-group">
                            <select v-model="form.type" name="type"  
                            class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                              <option value="">Select user role</option>
                              <option value="admin">Admin</option>
                              <option value="user">Standard User</option>
                              <option value="author">Author</option>
                            </select>
                           <has-error :form="form" field="type"></has-error>
                        </div>

                        <div class="form-group">
                          <input v-model="form.password" type="password" name="password" placeholder="Password" 
                            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                          <has-error :form="form" field="password"></has-error>
                        </div>
                  </div>
                  <div class="modal-footer">
                    <button type="button" class="btn btn-warning" data-dismiss="modal">Close</button>
                    <button type="submit" class="btn btn-primary">{{editmode ? 'Update User' : 'Create New User'}}</button>
                  </div>
              </form>
            </div>
          </div>
        </div>

    </div>

</template>

<script>
    export default {
        data() {

            return {
                editmode: false,
                users:{},
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
        methods: {
            getResults(page = 1) {
                axios.get('api/user?page=' + page)
                    .then(response => {
                        this.users = response.data;
                    });
            },
            updateUser(){
                this.$Progress.start();
                //console.log('editingdata')
                this.form.put('api/user/' + this.form.id)
                .then(() => {
                    $('#addNew').modal('hide');
                    Fire.$emit('AfterCreate');
                    toast.fire(
                                  'Data updated!',
                                  'Your file has been updated.',
                                  'success'
                                );
                    this.$Progress.finish();
                })
                .catch(() =>{
                    this.$Progress.fail();
                });
            },
            newModal(){
                this.editmode = false;
                this.form.reset();
                $('#addNew').modal('show');
            },
            editModal(user){
                this.editmode = true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user);
            },
            deleteUser(id){
                swal.fire({
                  title: 'Are you sure?',
                  text: "You won't be able to revert this!",
                  type: 'warning',
                  showCancelButton: true,
                  confirmButtonColor: '#3085d6',
                  cancelButtonColor: '#d33',
                  confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    //Send rqusert to server
                    //vform
                    if (result.value) {
                        this.form.delete('api/user/'+id).then(() => {
                            
                                toast.fire(
                                  'Deleted!',
                                  'Your file has been deleted.',
                                  'success'
                                );
                                Fire.$emit('AfterCreate');
                              
                          }).catch(() => {
                            swal("Failed!", "There is something wrong.", "warning");
                          });
                    }
                })
            },
            loadUsers() {
                if(this.$gate.isAdminOrAuthor()){
                    axios.get("api/user").then(({data}) => (this.users = data));
                }
                
            },
            submitUser() {
                this.$Progress.start();
                this.form.post('api/user')
                .then(() => {
                    Fire.$emit('AfterCreate');
                $('#addNew').modal('hide');

                toast.fire({
                  type: 'success',
                  title: 'User added successfuly!!!'
                });
                this.$Progress.finish(); 
                })
                .catch(() => {

                });
                 
            }
        },
        created() {
            Fire.$on('searching', () => {
                let query = this.$parent.search;
                axios.get('api/findUser?q=' + query)
                .then((data) => {
                    this.users = data.data;
                })
                .catch(() => {

                });
            });
           this.loadUsers();
           //setInterval(() => this.loadUsers(), 3000);
           Fire.$on('AfterCreate', () => {
            this.loadUsers();
           });
        }
    }
</script>