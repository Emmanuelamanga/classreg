
<template>
    <div class="container">
        <div class="row mt-5">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Registered Course Table</h3>
                <div class="card-tools">
                    <button class="btn btn-md btn-success" data-toggle="modal" data-target="#addNew" @click="openModalWindow">Add New Course<i class="fas fa-book-plus fa-fw"></i></button>
                </div>
              </div>

              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody>
                    <tr>
                        <th>ID</th>
                        <th>Course Name</th>
                        <th>Teacher Name</th>
                        <th>Total Hours</th>
                        <th>Registered At</th>
                        <th>Modify</th>
                  </tr>

                  <tr v-for="course in courses" :key="course.id">
                    <td>{{ course.id }}</td>
                    <td>{{ course.course_name | strToUpper}}</td>
                    <td>{{ course.teacher_name }}</td>
                    <td>{{ course.total_hours }}</td>
                    <td>{{ course.created_at | formatDate}}</td>
                    <td>
                        <a href="#" data-id="course.id" @click="editModalWindow(course)">
                            <span class="btn  btn-sm btn-warning">Edit</span>
                        </a>
                        |
                        <a href="#" @click="deleteCourse(course.id)">
                            <span class="btn  btn-sm btn-danger">Trash</span>

                        </a>

                    </td>
                  </tr>
                </tbody></table>
              </div>

              <div class="card-footer">

              </div>
            </div>

          </div>
        </div>


            <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                <div class="modal-header">

                    <h5 v-show="!editMode" class="modal-title" id="addNewLabel">Add New Course</h5>
                    <h5 v-show="editMode" class="modal-title" id="addNewLabel">Update Course</h5>

                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>

                <form @submit.prevent="editMode ? updateCourse() : createCourse()">
                <div class="modal-body">
                    <div class="form-group">
                        <input v-model="form.course_name" type="text" name="name"
                            placeholder="Course Name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('course_name') }">
                        <has-error :form="form" field="course_name"></has-error>
                    </div>

                    <div class="form-group">
                        <input v-model="form.teacher_name" type="text" name="teacher_name"
                            placeholder="Teacher's Name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('teacher_name') }">
                        <has-error :form="form" field="teacher_name"></has-error>
                    </div>


                    <div class="form-group">
                        <input v-model="form.total_hours" type="text" name="total_hours" id="total_hours" placeholder="Total Hours"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('total_hours') }">
                        <has-error :form="form" field="total_hours"></has-error>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                    <button v-show="editMode" type="submit" class="btn btn-primary">Update</button>
                    <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
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
                editMode: false,
                courses: {},
                form: new Form({
                    id: '',
                    name : '',
                    email: '',
                    password: '',

                })
            }
        },
        methods: {

        editModalWindow(course){
           this.form.clear();
           this.editMode = true
           this.form.reset();
           $('#addNew').modal('show');
           this.form.fill(course)
        },
        updateCourse(){
           this.form.put('api/course/'+this.form.id)
               .then(()=>{

                   Toast.fire({
                      icon: 'success',
                      title: 'course updated successfully'
                    })

                    Fire.$emit('AfterCreatedCourseLoadIt');

                    $('#addNew').modal('hide');
               })
               .catch(()=>{
                  console.log("Error.....")
               })
        },
        openModalWindow(){
           this.editMode = false
           this.form.reset();
           $('#addNew').modal('show');
        },

        loadCourses() {

        axios.get("api/course").then( data => (this.courses = data.data));

          //pick data from controller and push it into courses object

        },

        createCourse(){

            this.$Progress.start()

            this.form.post('api/course')
                .then(() => {

                    Fire.$emit('AfterCreatedCourseLoadIt'); //custom events

                        Toast.fire({
                          icon: 'success',
                          title: 'Course created successfully'
                        })

                        this.$Progress.finish()

                        $('#addNew').modal('hide');

                })
                .catch(() => {
                   console.log("Error......")
                })



            //this.loadCourses();
          },
          deleteCourse(id) {
            Swal.fire({
              title: 'Are you sure?',
              text: "You won't be able to revert this!",
              icon: 'warning',
              showCancelButton: true,
              confirmButtonColor: '#3085d6',
              cancelButtonColor: '#d33',
              confirmButtonText: 'Yes, delete it!'
            }).then((result) => {

              if (result.value) {
                //Send Request to server
                this.form.delete('api/course/'+id)
                    .then((response)=> {
                            Swal.fire(
                              'Deleted!',
                              'course deleted successfully',
                              'success'
                            )
                    this.loadCourses();

                    }).catch(() => {
                        Swal.fire({
                          icon: 'error',
                          title: 'Oops...',
                          text: 'Something went wrong!',
                          footer: '<a href>Why do I have this issue?</a>'
                        })
                    })
                }

            })
          }
        },

        created() {
            this.loadCourses();

            Fire.$on('AfterCreatedCourseLoadIt',()=>{ //custom events fire on
                this.loadCourses();
            });

        }
    }
</script>
