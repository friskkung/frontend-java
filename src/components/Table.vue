<template>
    <div>
        <v-data-table :headers="headers" :items="studentItem" sort-by="calories" class="elevation-1">
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
            <template v-slot:[`item.stdEmail`]="{ item }">{{ item.stdEmail }}</template>
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
                                <v-text-field v-model="stdId" label="รหัสนักศึกษา"></v-text-field>
                            </v-col>
                            <v-col cols="12" sm="6" md="6">
                                <v-text-field v-model="stdName" label="ชื่อ"></v-text-field>
                            </v-col>
                            <v-col cols="12" sm="6" md="6">
                                <v-text-field v-model="stdSurName" label="นามสกุล"></v-text-field>
                            </v-col>
                            <v-col cols="12" sm="6" md="6">
                                <v-text-field v-model="stdEmail" label="เมลล์"></v-text-field>
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
        stdId: '',
        stdName: '',
        stdSurName: '',
        stdEmail: '',
        headers: [
            {
                text: 'ไอดี',
                align: 'start',
                sortable: false,
                value: 'id',
            },
            { text: 'รหัสนักศึกษา', value: 'stdId' },
            { text: 'ชื่อ', value: 'stdName' },
            { text: 'นามสกุล', value: 'stdSurName' },
            { text: 'เมลล์', value: 'stdEmail' },
            { text: 'Actions', value: 'actions', sortable: false },
        ],
        studentItem: [],
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
        idStudent:'',
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
            this.studentItem = []
            try {
                var data = await this.axios.get('http://localhost:9001/students')
                console.log('data student===>', data)
                this.studentItem = data.data
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
                console.log("item.stdId===>",item.stdId)
                this.stdId = item.stdId
                console.log("stdId ===>",this.stdId)
                this.stdName = item.stdName
                this.stdSurName = item.stdSurName
                this.stdEmail = item.stdEmail
                this.idStudent = item.id

            }
        },
        deleteItem(item) {
            this.idForDelete = item.id
            this.dialogDelete = true
        },

        async deleteItemConfirm() {
            try {
                var data = await this.axios.delete('http://localhost:9001/students/'+this.idForDelete)
                this.initialize()
            } catch (error) {
                
            }
            this.closeDelete()
        },

        close() {
            this.dialog = false
            this.stdId=''
            this.stdName=''
            this.stdSurName=''
            this.stdEmail=''
        },

        closeDelete() {
            this.dialogDelete = false
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem)
                this.editedIndex = -1
            })
        },

        async save(actions) {
            var data = {
                    stdId: this.stdId,
                    stdName: this.stdName,
                    stdSurName: this.stdSurName,
                    stdEmail:this.stdEmail
                }
            if (actions === 'เพิ่มข้อมูล') {
                // this.desserts.push(this.editedItem)
                // this.dialog=false
                console.log('data before send===>', data)
                try {
                    console.log('dataResponse ===> ', dataResponse)
                    var dataResponse = await this.axios.post('http://localhost:9001/students', data)
                    console.log('dataResponse ===> ', dataResponse)
                    this.close()
                    this.initialize()
                } catch (error) {
                    console.log(error.message)
                }
            } else {
                console.log("item.stdId_save===>",this.stdId)
                console.log('data before send===>', data)
                try {
                    var dataResponse = await this.axios.put('http://localhost:9001/students/'+this.idStudent,data)
                    console.log('data after send ===>', dataResponse)
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