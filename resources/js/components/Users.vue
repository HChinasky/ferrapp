<template>
    <div class="container">
       <div class="row mt-5">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                  <button class="btn btn-success" @click="newModal">Add new 
                    <i class="fa fa-user-plus fa-fw"></i>
                  </button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody>
                    <tr>
                      <th>ID</th>
                      <th>Name</th>
                      <th>Email</th>
                      <th>Type</th>
                      <th>Register At</th>
                    </tr>
                    <tr v-for="user in users" :key="user.id">
                      <td>{{ user.id }}</td>
                      <td>{{ user.name | upText  }}</td>
                      <td>{{ user.email | upText  }}</td>
                      <td>{{ user.type | upText }}</td>
                      <td><span class="tag tag-success">{{ user.created_at |  myDate  }}</span></td>
                      <td>
                        <a href="#" @click="editModal(user)">
                          <i class="fa fa-edit blue"></i>
                        </a>
                        /
                        <a href="#" @click="deleteUser(user.id)">
                          <i class="fa fa-trash red"></i>
                        </a>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNew" aria-hidden="true">
          <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" v-show="!editmode" id="exampleModalLabel">Add new</h5>
                <h5 class="modal-title" v-show="editmode" id="exampleModalLabel">Update User's Info</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
              <form @submit.prevent="editmode ? updateUser() : createUser()">
                <div class="modal-body">

                  <div class="form-group">
                    <input v-model="form.name" type="text" name="name"
                      placeholder="Name"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                    <has-error :form="form" field="name"></has-error>
                  </div>

                  <div class="form-group">
                    <input v-model="form.email" type="email" name="email"
                      placeholder="Email"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                    <has-error :form="form" field="email"></has-error>
                  </div>

                  <div class="form-group">
                    <textarea  v-model="form.bio" type="text" name="bio"
                      placeholder="Short bio for user (Optional)"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                    <has-error :form="form" field="bio"></has-error>
                  </div>

                  <div class="form-group">
                    <select name="type" v-model="form.type" id="type"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                        <option value="">Select user role</option>
                        <option value="admin">Admin</option>
                        <option value="user">User</option>
                        <option value="author">Author</option>
                      </select>
                    <has-error :form="form" field="type"></has-error>
                  </div>

                  <div class="form-group">
                    <input v-model="form.password" type="password" name="password"
                      placeholder="Password"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                    <has-error :form="form" field="password"></has-error>
                  </div>

                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                  <button type="sumbit" v-show="editmode" class="btn btn-success">Update</button>
                  <button type="sumbit" v-show="!editmode" class="btn btn-primary">Create</button>
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
        users : {},
        form: new form({
          id: '',
          name : '',
          email : '',
          password : '',
          type : '',
          bio : '',
          photo : ''
        })
      }
    },
    methods: {
      updateUser(id) {
        this.$Progress.start();
        this.form.put('api/user/'+this.form.id)
        .then(() => {
          $('#addNew').modal('hide');
          Swal.fire(
                    'Updated!',
                    'User info has been updated.',
                    'success'
                  )
          this.$Progress.finish(); 
          Fire.$emit('AfterCreated');
        })
        .catch(() => {
          this.$Progress.fail();  
          Toast.fire({
              type: 'error',
              title: 'Update user info failed!'
            });        
        });
      },
      editModal(user) {
        this.editmode = true;
        this.form.reset();
        $('#addNew').modal('show');
        this.form.fill(user);
      },
      newModal() {
        this.editmode = false;
        this.form.reset();
        $('#addNew').modal('show');
      },
      deleteUser(id) {
        Swal.fire({
          title: 'Are you sure?',
          text: "You won't be able to revert this!",
          type: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Yes, delete it!'
        }).then((result) => {
            if (result.value) {
              this.form.delete('api/user/'+id).then(() => {
                  Swal.fire(
                    'Deleted!',
                    'Your file has been deleted.',
                    'success'
                  )
                  Fire.$emit('AfterCreated');
              }).catch(() => {
                Swal.fire("Failed!", "There was something wronge", "warning");
              });
            }
          })
        },
        loadUsers() {
          axios.get('api/user').then(({ data }) => (this.users = data.data));
        },
        createUser() {
          this.$Progress.start();
          this.form.post('api/user')
          .then(() => {
            Fire.$emit('AfterCreated');
            $('#addNew').modal('hide');
            Toast.fire({
              type: 'success',
              title: 'Created user successfully!'
            });

          this.$Progress.finish();            
          })
          .catch((e) => {
            console.log('Error: ' + e);
          });
        
        }
      },
        created() {
            this.loadUsers();
            Fire.$on('AfterCreated',() => {
              this.loadUsers();
            });
            // setInterval(() => this.loadUsers(), 3000);
        }
    }
</script>
