<template>
    <div class="container">
        <div class="row" v-if="$gate.isAdminOrAuthor()">
          <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Responsive Hover Table</h3>
                <div class="card-tools">
                  <button type="button"  class="btn btn-success btn-circle btn-xl" @click="newModal">
                        <i class="fas fa-user-plus"></i>
                  </button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover text-nowrap">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>Name</th>
                      <th>Email</th>
                      <th>Type</th>
                      <th>Registered At</th>
                      <th>Modify</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="user in users.data" :key="user.id">
                      <td>{{user.id}}</td>
                      <td>{{user.name}}</td>
                      <td>{{user.email}}</td>
                      <td><span class="tag tag-success">{{user.type | upText}}</span></td>
                      <td>{{user.created_at | formateDate}}</td>
                    <td width="10">
                         <!-- <a class="btn btn-primary btn-sm" href="#">
                              <i class="fas fa-folder">
                              </i>
                              View
                          </a>-->
                          <a class="btn btn-info btn-sm" href="" @click.prevent="editModal(user)">
                              <i class="fas fa-pencil-alt">
                              </i>
                              Edit
                          </a>
                          <a class="btn btn-danger btn-sm" href="" @click.prevent="deleteUser(user.id)">
                              <i class="fas fa-trash">
                              </i>
                              Delete
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
            </div>
            <!-- /.card -->
          </div>
        </div>
        <div v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
          </div>
        <!-- Modal -->
        <div class="modal fade" id="exampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
          <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" v-show="!editmode" id="exampleModalLabel">Add New User</h5>
                <h5 class="modal-title" v-show="editmode" id="exampleModalLabel">Update User Info</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
                <form @submit.prevent="editmode? updateUser(): createUser()" id="createFormSubmit">
                    <div class="modal-body">
                        <div class="form-group">
                          <label>Nmae</label>
                          <input v-model="form.name" type="text" name="name" placeholder="Name" 
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                          <has-error :form="form" field="name"></has-error>
                        </div>
                        <div class="form-group">
                          <label>Email</label>
                          <input v-model="form.email" type="email" name="email" placeholder="Email" 
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                          <has-error :form="form" field="email"></has-error>
                        </div>
                         <div class="form-group">
                            <textarea v-model="form.bio" name="bio" id="bio"
                            placeholder="Short bio for user (Optional)"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                            <has-error :form="form" field="bio"></has-error>
                        </div>

                        <div class="form-group">
                            <select name="type" v-model="form.type" id="type" class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                <option value="">Select User Role</option>
                                <option value="admin">Admin</option>
                                <option value="user">Standard User</option>
                                <option value="author">Author</option>
                            </select>
                            <has-error :form="form" field="type"></has-error>
                        </div>

                        <div class="form-group">
                          <label>Password</label>
                          <input v-model="form.password" type="password" name="password" placeholder="Password"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                          <has-error :form="form" field="password"></has-error>
                        </div>
                   
                    </div>
                      <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="submit" v-show="editmode" class="btn btn-success">Update</button>
                        <button type="submit" v-show="!editmode" class="btn btn-primary">Create</button>
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
                users: {},
                form: new Form({
                    id : '',
                    name : '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                })
            }
        },
        methods: {
            loadUsers(){
              if (this.$gate.isAdminOrAuthor()) {
               // axios.get("api/user").then(({ data }) => (this.users = data.data));
                axios.get("api/user").then(({ data }) => (this.users = data));
              }
            },
            editModal(user){
                this.editmode = true;
                this.form.reset();
                this.form.clear();
                $('#exampleModal').modal('show');
                this.form.fill(user);
            },
            newModal(){
                this.editmode = false;
                this.form.reset();
                this.form.clear();
                $('#exampleModal').modal('show');
            },
            createUser () {
              //$('#createFormSubmit')[0].reset();
              // Submit the form via a POST request
              this.$Progress.start();
              this.form.post('api/user')
                .then(({ data }) => { 
                    Fire.$emit('reloadData');
                    $('#exampleModal').modal('hide');
                    $('.modal-backdrop').remove();
                    toast.fire({
                      icon: 'success',
                      title: 'User created successfully'
                    });
                    this.$Progress.finish();
                    //console.log(data) 
                })
                .catch(({err}) => {
                    this.$Progress.fail();
                })
            },
            updateUser(){
              this.$Progress.start();
              this.form.put('api/user/'+this.form.id)
              .then(()=>{
                  $('#exampleModal').modal('hide');
                  $('.modal-backdrop').remove();
                  swal.fire(
                  'Updated!',
                  'Information has been updated.',
                  'success'
                  )
                  this.$Progress.finish();
                  Fire.$emit('reloadData');
              })
              .catch(()=>{
                this.$Progress.fail();
              })
            },
            deleteUser(id){
               swal.fire({
                  title: 'Are you sure?',
                  text: "You won't be able to revert this!",
                  icon: 'warning',
                  showCancelButton: true,
                  confirmButtonColor: '#3085d6',
                  cancelButtonColor: '#d33',
                  confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                  if (result.value) {
                    this.form.delete('api/user/'+id).then(()=>{
                        swal.fire(
                      'Deleted!',
                      'User data has been deleted.',
                      'success'
                    )
                    Fire.$emit('reloadData');
                    }).catch(()=> {
                       swal.fire({
                        icon: 'warning',
                        title: 'Warning...',
                        text: 'You have not permission to delete it',
                      })
                    });
                    
                  }
                })
            },
            // Our method to GET results from a Laravel endpoint
          getResults(page = 1) {
            axios.get('api/user?page=' + page)
              .then(response => {
                this.users = response.data;
              });
          }
        },
        created() {
              this.loadUsers();
              Fire.$on('reloadData',()=>{
                this.loadUsers();
              });
              //setInterval(()=>{this.loadUsers()},3000);
              Fire.$on('searching',()=>{
                let query=this.$parent.search;
                axios.get('api/findUser?q='+query)
                .then((data)=>{
                  this.users=data.data;
                })
                .catch(()=>{

                })
              });
        }
    }
</script>
