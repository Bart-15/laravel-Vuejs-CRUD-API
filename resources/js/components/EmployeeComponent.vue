<template>
    <div class="container employee">
        <h4 class="text-center">Laravel + Vue JS CRUP API</h4>
        <div class="row justify-content-center mt-5">
            <div class="col-sm-12 col-md-12 col-lg-12">
                <div class="card" tyle="width:24rem;">
                    <div class="card-tools">
                        <button
                            class="btn btn-success float-right m-3"
                            @click="addModal()"
                        >
                            +Add new
                        </button>
                    </div>
                    <div class="card-body">
                        <table class="table">
                            <thead class="thead-dark">
                                <tr>
                                    <th scope="col">#</th>
                                    <th scope="col">First</th>
                                    <th scope="col">Lastname</th>
                                    <th scope="col">Created</th>
                                    <th>Actions</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="employee in employees.data" :key="employee.id">
                                    <th scope="row">{{employee.id}}</th>
                                    <td>{{employee.firstName | upperFirst}}</td>
                                    <td>{{employee.lastName | upperFirst}}</td>
                                    <td>{{employee.created_at | myDate}}</td>
                                    <td>
                                        <a id="edit" href="" @click.prevent="editUserModal(employee)">
                                            <i class="fas fa-edit fa-lg"></i>
                                        </a>
                                        <a href="" class="text-danger"  @click.prevent="deleteEmployee(employee.id)">
                                            <i class="fa fa-trash red fa-lg" aria-hidden="true"></i>
                                        </a>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <!-- Modal Start -->
        <!-- Modal -->
        <div
            class="modal fade"
            id="addEmployee"
            tabindex="-1"
            role="dialog"
            aria-labelledby="addEmployeeTitle"
            aria-hidden="true"
        >
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="exampleModalLongTitle">
                            <h4 v-show="!editMode">Add Employee</h4>
                            <h4 v-show="editMode">Update Employee</h4>
                        </h5>
                        <button
                            type="button"
                            class="close"
                            data-dismiss="modal"
                            aria-label="Close"
                        >
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <form @submit.prevent="editMode ? updateEmployee() : addEmployee()">
                            <div class="form-group">
                                <input type="text" name="firstName" v-model="form.firstName" class="form-control" :class="{ 'is-invalid': form.errors.has('firstName') }" placeholder="Firstname">
                                 <has-error :form="form" field="firstName"></has-error>
                            </div>
                             <div class="form-group">
                                <input type="text" name="lastName" v-model="form.lastName" :class="{ 'is-invalid': form.errors.has('lastName') }" class="form-control" placeholder="Lastname">
                                 <has-error :form="form" field="lastName"></has-error>
                            </div>
                            <div class="form-group">
                                <button type="submit" v-show="!editMode" class="btn btn-primary float-right">Add Employee</button>
                                <button type="submit" v-show="editMode" class="btn btn-success float-right">Update Employee</button>
                            </div>
                        </form>
                    </div>
                    <div class="modal-footer">
                        <!-- <button
                            type="button"
                            class="btn btn-secondary"
                            data-dismiss="modal"
                        >
                            Close
                        </button>
                        <button type="button" class="btn btn-primary">
                            Save changes
                        </button> -->
                    </div>
                </div>
            </div>
        </div>
        <!-- End modal -->
    </div>
</template>

<script>
import $ from "jquery";
import Form from 'vform'
import Swal from 'sweetalert2'
import axios from 'axios'
    const Toast = Swal.mixin({ //when firing the toast, the first window closes automatically
      toast: true,
      position: 'top-end',
      showConfirmButton: false,
      timer: 3000
    });
export default {
    data() {
        return {
            employees:{},
            form: new Form({
                id:'',
                firstName:'',
                lastName:''
            }),
            editMode:false
        };
    },

    methods: {
        addModal() {
            $("#addEmployee").modal("show");
            this.form.reset();
            this.editMode = false
        },

        //Add Employee
         addEmployee() {
            this.form.post('api/employee')
            .then(() => {
                 Toast.fire({
                     icon:'success',
                  title: 'Employee Added Successfully!'
                })
                $("#addEmployee").modal("hide")
                Fire.$emit('afterCreated')
            })
         },

         //Fetch all Employee
         loadEmployee() {
             axios.get('api/employee')
             .then(({data}) => (this.employees = data))
         },

        //Open Modal for edit function
        editUserModal(employee){
             $("#addEmployee").modal("show")
             this.form.fill(employee)
             this.editMode = true
        },

        //Delete Employee
        deleteEmployee(id){
             Swal.fire({
                  title: 'Are you sure?',
                  text: "You won't be able to revert this!",
                  icon: 'warning',
                  showCancelButton: true,
                  confirmButtonColor: '#3085d6',
                  cancelButtonColor: '#d33',
                  confirmButtonText: 'Yes, delete it!'
                })
            .then((result) => {

                //Send request to API
                if(result.value) {
                    this.form.delete('api/employee/'+id)
                    .then(() => {
                        Swal.fire(
                          'Deleted!',
                          'Your file has been deleted.',
                          'success'
                          )
                          Fire.$emit('afterCreated')
                    })
                    .catch(() => {
                          Swal.fire("Failed!", "There was something wrong.", "warning");
                    })
                }
            })    
        },

        //Update Employee
        updateEmployee(id) {
            this.form.put('api/employee/'+this.form.id)
            //If Success then proceed
            .then(() => {
                 Toast.fire({
                    icon:'success',
                    title: 'Employee Updated Successfully!'
                })
                Fire.$emit('afterCreated');
                $('#addEmployee').modal("hide")
                this.form.reset();
            })
        }
    },

    created() {
        this.loadEmployee();
        Fire.$on('afterCreated', () => {
            this.loadEmployee();
        })
    }
};
</script>

<style></style>
