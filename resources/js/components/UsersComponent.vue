<template>
<div class ="container">  
      <div class="row mt-5">
          <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users List</h3>

<div class="card-tools">
                 
<button type="button" class="btn btn-success" @click="newModal">
  Add New User <i class="fas fa-user-plus"></i>
</button>

<!-- Modal -->
<div class="modal fade" id="addUser" tabindex="-1" role="dialog"  aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" v-show="!editMode" id="addUserLabel">Add User</h5>
        <h5 class="modal-title" v-show="editMode" id="addUserLabel">Update User</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
</div>
  <form @submit.prevent="editMode ? updateUser() : createUser()" >
    <div class="modal-body">   
   <div class="form-group">
      <input v-model="form.name" type="text" name="name" placeholder="Enter Name"
        class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
      <has-error :form="form" field="name"></has-error>
   </div>

   <div class="form-group">
      <input v-model="form.email" type="email" name="email" placeholder="Enter Email"
        class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
      <has-error :form="form" field="email"></has-error>
   </div>

   

   <div class="form-group">
      <select name="type" v-model="form.type" id="type" 
        class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
      <option value="">Select User Role</option>
      <option value="admin">Admin</option>
      <option value="user">Standard User</option>
      <option value="author">Author</option>
      </select>
      <has-error :form="form" field="type"></has-error>
   </div>

   <div class="form-group">
      <textarea v-model="form.bio" type="text" name="bio" placeholder="Short Bio for User (Optional)"
        class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
      <has-error :form="form" field="bio"></has-error>
   </div>

   <div class="form-group">
      <input v-model="form.password" type="password" name="password" placeholder="Enter Password"
        class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
      <has-error :form="form" field="password"></has-error>
   </div>

  </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
        <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
        <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
      </div>
      </form>
    </div>
  </div>
</div>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover text-nowrap">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>NAME</th>
                      <th>EMAIL</th>
                      <th>TYPE</th>
                      <th>REGISTERED</th>
                      <th>MODIFY</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="user in users" :key="user.id">
                      <td>{{user.id}}</td>
                      <td>{{user.name}}</td>
                      <td>{{user.email}}</td>
                      <td>{{user.type | upText}}</td>
                      <td>{{user.created_at |myDate}}</td>
                     <td><a href="#" @click="editModal(user)"><i class="fa fa-edit"></i></a>
                       /                      
                      <a href="#" @click="deleteUser(user.id)">
                      <i class="fa fa-trash" style="color:#FF0000;"></i></a></td>
                    </tr>
                    
                  </tbody>
                </table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
        </div>
        <!-- /.row -->
        </div>
</template>

<script>
    export default {
      data() {
        return{
          editMode:false,
          users:{},
          form: new Form({
            id: '',
            name: '',
            email: '',
            password: '',
            type: '',
            bio: '',
            photo: ''
          })
        }

      },
      methods:{
        updateUser(){
         this.$Progress.start();       
         this.form.put('api/user/'+this.form.id)
         .then(() =>{
             $('#addUser').modal('hide');
             Swal.fire(
                        'Updated!',
                        'Your file has been updated.',
                         'success'
                     )
            this.$Progress.finish();
            Fire.$emit('AfterCreate');                     

         })
         .catch(() =>{
            this.$Progress.fail();  
         });
        },
        newModal(){
          this.editMode=false;
           this.form.reset();
           $('#addUser').modal('show');

        },
        editModal(user){
          this.editMode = true;
           this.form.reset();
           $('#addUser').modal('show');
           this.form.fill(user);

        },
        deleteUser(id){
                      Swal.fire({
                        title: 'Are you sure?',
                        text: "You won't be able to revert this!",
                        icon: 'warning',
                        showCancelButton: true,
                        confirmButtonColor: '#3085d6',
                        cancelButtonColor: '#d33',
                        confirmButtonText: 'Yes, delete it!'
                      }).then((result) => {

                        //Send http request to server
                        this.form.delete('api/user/'+id).then(()=>{
                        if (result.value) {
                          Swal.fire(
                            'Deleted!',
                            'Your file has been deleted.',
                            'success'
                          )
                          Fire.$emit('AfterCreate');
                        }
                        }).catch(()=>{
                          Swal("Failed" ,"Something went wrong", "warning");
                        });
                      })
        },
        loadUsers(){
          axios.get('api/user').then(({data})=>(this.users = data.data));

        },
           createUser(){
             this.$Progress.start()
             this.form.post('api/user')
             .then(()=>{ Fire.$emit('AfterCreate');
            $('#addUser').modal('hide')
             Toast.fire({
                        icon: 'success',
                        title: 'User Created successfully'
                      })
              this.$Progress.finish();              
              })
              .catch(()=>{
                
              })
            
           },
          },
            created() {
                  this.loadUsers();
                  Fire.$on('AfterCreate',() => {this.loadUsers()});
                     }
        }
    
        </script>
