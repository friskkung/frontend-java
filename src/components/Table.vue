<template>
    <div>
        <v-data-table :headers="headers" :items="employeeItem" sort-by="calories" class="elevation-1">
            <template v-slot:top>
                <v-toolbar flat>
                    <v-toolbar-title>จัดการข้อมูล</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-spacer></v-spacer>
                    <v-btn color="primary" dark class="mb-2" @click="openDialog('add', defaultItem)">
                        เพิ่มข้อมูล
                    </v-btn>
                </v-toolbar>
            </template>
            <template v-slot:[`item.role`]="{ item }">{{ item.role.name }}</template>
            <template v-slot:[`item.actions`]="{ item }">
                <v-icon small class="mr-2" @click="openDialog('แก้ไขข้อมูล', item)">
                    mdi-pencil
                </v-icon>
                <v-icon small @click="deleteItem(item)">
                    mdi-delete
                </v-icon>
            </template>
            <template v-slot:no-data>
                <v-btn color="primary" @click="initialize">
                    Reset
                </v-btn>
            </template>
        </v-data-table>
        <v-dialog v-model="dialog" max-width="500px">
            <v-card>
                <v-card-title>
                    <span class="text-h5">{{ formTitle }}</span>
                </v-card-title>

                <v-card-text>
                    <v-container>
                        <v-row>
                            <v-col cols="12" sm="6" md="6">
                                <v-text-field v-model="firstName" label="ชื่อ"></v-text-field>
                            </v-col>
                            <v-col cols="12" sm="6" md="6">
                                <v-text-field v-model="lastName" label="นามสกุล"></v-text-field>
                            </v-col>
                            <v-col cols="12" sm="6" md="6">
                                <v-text-field v-model="salary" label="เงินเดือน"></v-text-field>
                            </v-col>
                            <v-col cols="12" sm="6" md="6">
                                <v-text-field v-model="role" label="ตำแหน่ง"></v-text-field>
                            </v-col>
                        </v-row>
                    </v-container>
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="close">
                        Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="save(formTitle)">
                        บันทึกข้อมูล
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
                <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                    <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
                    <v-spacer></v-spacer>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </div>
</template>

<script>
export default {
    data: () => ({
        dialog: false,
        dialogDelete: false,
        firstName: '',
        lastName: '',
        salary: '',
        role: '',
        headers: [
            {
                text: 'ไอดี',
                align: 'start',
                sortable: false,
                value: 'employeeId',
            },
            { text: 'ชื่อ', value: 'firstName' },
            { text: 'นามสกุล', value: 'lastName' },
            { text: 'เงินเดือน', value: 'salary' },
            { text: 'ตำแหน่ง', value: 'role' },
            { text: 'Actions', value: 'actions', sortable: false },
        ],
        employeeItem: [],
        editedIndex: -1,
        editedItem: {
            name: '',
            calories: 0,
            fat: 0,
            carbs: 0,
            protein: 0,
        },
        defaultItem: {
            name: '',
            calories: 0,
            fat: 0,
            carbs: 0,
            protein: 0,
        },
        formTitle: '',
        idEmployee:'',
        idForDelete:''
    }),
    watch: {
        dialog(val) {
            val || this.close()
        },
        dialogDelete(val) {
            val || this.closeDelete()
        },
    },

    created() {
        this.initialize()
    },

    methods: {
        async initialize() {
            this.employeeItem = []
            try {
                var data = await this.axios.get('http://localhost:9001/employee')
                console.log('data employee===>', data)
                this.employeeItem = data.data
            } catch (error) {
                console.log(error.message)
            }
        },
        openDialog(Action, item) {
            //console.log(Action, item)
            this.formTitle = ''
            if (Action === 'add') {
                this.formTitle = 'เพิ่มข้อมูล'
                this.editItem = item
                this.dialog = true
            } else {
                this.formTitle = 'แก้ไขข้อมูล'
                this.dialog = true
                this.firstName = item.firstName
                this.lastName = item.lastName
                this.salary = item.salary
                this.role = item.role.name
                this.idEmployee = item.employeeId

            }
        },
        deleteItem(item) {
            this.idForDelete = item.employeeId
            this.dialogDelete = true
        },

        async deleteItemConfirm() {
            try {
                var data = await this.axios.delete('http://localhost:9001/employee/'+this.idForDelete)
                this.initialize()
            } catch (error) {
                
            }
            this.closeDelete()
        },

        close() {
            this.dialog = false
            this.firstName=''
            this.lastName=''
            this.salary=''
            this.role=''
        },

        closeDelete() {
            this.dialogDelete = false
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem)
                this.editedIndex = -1
            })
        },

        async save(actions) {
            if (actions === 'เพิ่มข้อมูล') {
                // this.desserts.push(this.editedItem)
                // this.dialog=false
                var data = {
                    firstName: this.firstName,
                    lastName: this.lastName,
                    salary: this.salary,
                    role: {
                        name:this.role
                    },
                    skills: [
                        {
                            skill:''
                        }
                    ]
                }
                console.log('data after send===>', data)
                try {
                    var dataResponse = await this.axios.post('http://localhost:9001/employee', data)
                    console.log('dataResponse ===>', dataResponse)
                    this.close()
                    this.initialize()
                } catch (error) {
                    console.log(error.message)
                }
            } else {
                var data = {
                    firstName: this.firstName,
                    lastName: this.lastName,
                    salary: this.salary,
                    role: {
                        name:this.role
                    },
                    skills: [
                        {
                            skill:''
                        }
                    ]
                }
                console.log('data after send===>', data)
                try {
                    var dataResponse = await this.axios.put('http://localhost:9001/employee/'+this.idEmployee,data)
                    console.log('dataResponse ===>', dataResponse)
                    this.close()
                    this.initialize()
                } catch (error) {
                    console.log(error.message)
                }
            }
            this.close()
        },
    },
}
</script>

<style></style>