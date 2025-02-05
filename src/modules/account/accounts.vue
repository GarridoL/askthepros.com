<template>
  <div class="ledger-summary-container">
    <div class="incre-row">
      <h2> Account Management </h2>
    </div>
    <basic-filter 
      v-bind:category="category" 
      :activeCategoryIndex="0"
      :activeSortingIndex="0"
      @changeSortEvent="retrieve($event.sort, $event.filter)"
      @changeStyle="manageGrid($event)"
      :grid="['list', 'th-large']"
      :sortByStyle="{
        background: '#01004E !important',
        color: 'white'
      }"
      :dropDownStyle="{
        border: '1px solid #01004E !important',
        background: 'none !important',
        color: '#272727 !important',
        borderLeft: '0px',
        outline: 'none !important',
        boxShadow: 'none !important'
      }"
    />
    
    <table class="table table-borderless table-responsive" v-if="data.length > 0">
      <thead>
        <tr>
          <th scope="col">Date</th>
          <th scope="col">Username</th>
          <th scope="col">Email Address</th>
          <th scope="col">Name</th>
          <th scope="col">Business Name</th>
          <th scope="col">Contact Number</th>
          <th scope="col">Type</th>
          <th scope="col">Industry</th>
          <th scope="col">Status</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in returnData" :key="index">
          <td>{{item.created_at}}</td>
          <td>
            <label class="action-link text-primary">{{item.username}}</label>
          </td>
          <td>{{item.email}}</td>
          <td v-if="!item.first_name && !item.last_name">NOT_SET</td>
          <td v-if="item.first_name && !item.last_name">{{item.first_name}}</td>
          <td v-if="!item.first_name && item.last_name">{{item.last_name}}</td>
          <td v-if="item.first_name && item.last_name">{{item.first_name + ' ' + item.last_name}}</td>
          <td>{{item.business_name ? item.business_name : 'NOT_SET'}}</td>
          <td>{{item.cellular_number ? item.cellular_number : 'NOT_SET'}}</td>
          <td>
            <label v-if="editTypeIndex !== index">{{item.account_type}}</label>
            <i class="fa fa-pencil text-primary" style="margin-left: 10px;" @click="setEditTypeIndex(index, item)" v-if="editTypeIndex !== index"></i>
            <span v-if="editTypeIndex === index">
              <select class="form-control" v-model="newAccountType" style="float: left; width: 70%;">
                <option v-if="user.type === 'ADMIN'" v-for="(typeItem, typeIndex) in ['USER', 'EXPERT', 'ADMIN']" :key="typeIndex">{{typeItem}}</option>
                <option v-else v-for="(typeItem, typeIndex) in ['USER', 'EXPERT', 'ADMIN']" :key="typeIndex">{{typeItem}}</option>
              </select>
              <i class="fa fa-check text-primary" style="margin-left: 5px; float: left;" @click="updateType(item, index)"></i>
              <i class="fa fa-times text-danger" style="margin-left: 5px; float: left;" @click="setEditTypeIndex(index, item)"></i>
            </span>
          </td>
          <td>
            <label v-if="editIndustryIndex !== index && item.account_type !== 'EXPERT'">{{JSON.parse(item.industry) ? JSON.parse(item.industry).industry : ''}}</label>
            <!-- <label v-if="editIndustryIndex !== index && item.account_type !== 'EXPERT'">{{Object.values(JSON.parse(item.industry))[0]}}</label> -->
            <label v-if="editIndustryIndex !== index && item.account_type === 'EXPERT'">{{expertIndustry.length >= 1 ? expertIndustry : 'No Industry'}}</label>
            <i class="fa fa-pencil text-primary" style="margin-left: 10px;" @click="setEditIndustryIndex(index, item)" v-if="editIndustryIndex !== index && item.account_type === 'EXPERT'"></i>
            <span v-if="editIndustryIndex === index">
              <select class="form-control" v-model="newIndustry" style="float: left; width: 70%;">
                <option v-if="user.type === 'ADMIN'" v-for="(typeItem, typeIndex) in industry" :key="typeIndex">{{typeItem}}</option>
                <option v-else v-for="(typeItem, typeIndex) in industry" :key="typeIndex">{{typeItem}}</option>
              </select>
              <i class="fa fa-check text-primary" style="margin-left: 5px; float: left;" @click="updateIndustry(item, index)"></i>
              <i class="fa fa-times text-danger" style="margin-left: 5px; float: left;" @click="setEditIndustryIndex(index, item)"></i>
            </span>
          </td>
          <td>{{item.status}}</td>
        </tr>
      </tbody>
    </table>

    <Pager
      :pages="numPages"
      :active="activePage"
      :limit="limit"
      v-if="data.length > 0"
    />

    <empty v-if="data.length <= 0" :title="'No accounts available!'" :action="'Keep growing.'"></empty>
  </div>
</template>
<style scoped>
.py-3{
  padding-top: 0 !important;
  padding-bottom: 0 !important;
}
.ledger-summary-container{
  width: 100%;
  float: left;
  height: auto;
  margin-bottom: 100px;
  margin-top: 25px;
  min-height: 70vh;
}

td i {
  padding-right: 0px !important;
  padding-left: 0px !important;
}

td {
border-bottom: 0.5px solid #e0e0e0;
}

@media (max-width: 992px){
  .ledger-summary-container{
    width: 100%;
  }
}

.card{
  margin:2%
}

</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import CONFIG from 'src/config.js'
import Pager from 'src/components/increment/generic/pager/Pager.vue'
export default{
  mounted(){
    $('#loading').css({display: 'block'})
    this.retrieve({created_at: 'desc'}, {column: 'created_at', value: ''})
    this.retrievePayloads()
    this.retrieveExpertPayloads()
  },
  computed: {
    returnData(){
      return this.data
    }
  },
  data(){
    return {
      user: AUTH.user,
      data: [],
      data1: [],
      data2: [],
      auth: AUTH,
      selecteditem: null,
      config: CONFIG,
      limit: 5,
      offset: 0,
      numPages: null,
      category: [{
        title: 'Sort by',
        sorting: [{
          title: 'Created ascending',
          payload: 'created_at',
          payload_value: 'asc',
          type: 'date'
        }, {
          title: 'Created descending',
          payload: 'created_at',
          payload_value: 'desc',
          type: 'date'
        }, {
          title: 'Username ascending',
          payload: 'username',
          payload_value: 'asc',
          type: 'text'
        }, {
          title: 'Username descending',
          payload: 'username',
          payload_value: 'desc',
          type: 'text'
        }, {
          title: 'Email ascending',
          payload: 'email',
          payload_value: 'asc',
          type: 'text'
        }, {
          title: 'Email descending',
          payload: 'email',
          payload_value: 'desc',
          type: 'text'
        }, {
          title: 'Type ascending',
          payload: 'account_type',
          payload_value: 'asc',
          type: 'text'
        }, {
          title: 'Type descending',
          payload: 'account_type',
          payload_value: 'desc',
          type: 'text'
        }, {
          title: 'Status ascending',
          payload: 'status',
          payload_value: 'asc',
          type: 'text'
        }, {
          title: 'Status descending',
          payload: 'status',
          payload_value: 'desc',
          type: 'text'
        }]
      }],
      filter: null,
      sort: null,
      editTypeIndex: null,
      editIndustryIndex: null,
      newAccountType: null,
      newIndustry: null,
      selectedAccount: null,
      selectedIndustry: null,
      activeItem: 'home',
      activePage: 1,
      location: null,
      industry: [],
      expertIndustry: [],
      selectIndust: false
    }
  },
  components: {
    'empty': require('components/increment/generic/empty/Empty.vue'),
    'basic-filter': require('src/components/increment/generic/filter/FilterWithCalendar.vue'),
    Pager

  },
  methods: {
    setEditTypeIndex(index, item){
      if(index === this.editTypeIndex){
        this.editTypeIndex = null
        this.newAccountType = null
      }else{
        this.selectedAccount = item
        this.editTypeIndex = index
        this.newAccountType = item.account_type
      }
    },
    setEditIndustryIndex(index, item){
      if(index === this.editIndustryIndex){
        this.editIndustryIndex = null
        this.newIndustry = null
      }else{
        this.selectedIndustry = item
        this.editIndustryIndex = index
        this.newIndustry = item.account_type
      }
    },
    updateType(item, index){
      if(this.newAccountType === null || this.newAccountType === item.account_type){
        this.setEditTypeIndex(index, item)
        return
      }
      let parameter = {
        id: item.id,
        account_type: this.newAccountType
      }
      $('#loading').css({display: 'block'})
      this.APIRequest('accounts/update_account_type', parameter).then(response => {
        $('#loading').css({display: 'none'})
        if(this.newAccountType === 'EXPERT'){
          this.updateIndustry(item, index)
          this.selectIndust = true
        }
        this.setEditTypeIndex(index, item)
        this.retrieve(null, null)
      })
    },
    updateIndustry(item, index){
      if(this.newIndustry === null && this.selectIndust === true){
        this.industry.map(ndx => {
          if(ndx === JSON.parse(item.industry).industry){
            this.editIndustryIndex = index
            this.newIndustry = ndx
          }
        })
      }else if(this.selectIndust === false){
        if(this.newIndustry === null){
          this.setEditIndustryIndex(index, item)
          return
        }
      }
      let parameter = {
        account_id: item.id,
        payload: 'assigned_industry',
        payload_value: this.newIndustry
      }
      $('#loading').css({display: 'block'})
      this.APIRequest('payloads/create_industry', parameter).then(response => {
        $('#loading').css({display: 'none'})
        if(response.data >= 1){
          this.setEditIndustryIndex(index, item)
          this.retrieve(null, null)
        }
      })
    },
    showProfileModal(item){
      this.selecteditem = item
      this.$children[1].$children[0].retrieveLocation(item)
      this.$children[1].$children[0].retrieveImage(item)
      this.selecteditem['payload'] = 'account'
      this.$children[1].$children[0].$children[0].retrieveRatings(item)
      // this.$children[1].$children[0].$children[0].retrieveRatings()
      $('#profileModal').modal('show')
    },
    redirect(params){
      ROUTER.push(params)
    },
    retrieve(sort, filter){
      if(sort !== null){
        this.sort = sort
      }
      if(filter !== null){
        this.filter = filter
      }
      if(sort === null && this.sort !== null){
        sort = this.sort
      }
      if(filter === null && this.filter !== null){
        filter = this.filter
      }
      let parameter = {
        condition: [{
          value: filter.value !== null ? '%' + filter.value + '%' : '%%',
          column: filter.column,
          clause: 'like'
        }],
        sort: sort,
        limit: this.limit,
        offset: (this.activePage > 0) ? ((this.activePage - 1) * this.limit) : this.activePage
      }
      $('#loading').css({display: 'block'})
      this.APIRequest('users/retrieve', parameter).then(response => {
        $('#loading').css({display: 'none'})
        if(response.data.length > 0){
          this.retrieveExpertPayloads()
          this.data = response.data
          this.numPages = parseInt(response.size / this.limit) + (response.size % this.limit ? 1 : 0)
        }else{
          this.data = []
          this.numPages = null
        }
      })
    },
    retrievePayloads(){
      let conditions = [{
        value: 'subscriptions',
        clause: '=',
        column: 'payload'
      }]
      let parameter = {
        condition: conditions
      }
      $('#loading').css({'display': 'block'})
      this.APIRequest('payloads/retrieve', parameter).then(response => {
        $('#loading').css({'display': 'none'})
        if(response.data.length > 0) {
          response.data.map(el => {
            this.industry.push(el.category)
            return el.category
          })
        }else{
          this.industry = []
        }
      }).catch(error => {
        $('#loading').css({'display': 'none'})
        error
      })
    },
    retrieveExpertPayloads(){
      let conditions = [{
        value: 'assigned_industry',
        clause: '=',
        column: 'payload'
      }]
      let parameter = {
        condition: conditions
      }
      $('#loading').css({'display': 'block'})
      this.APIRequest('payloads/retrieve', parameter).then(response => {
        $('#loading').css({'display': 'none'})
        if(response.data.length > 0) {
          response.data.map(elI => {
            this.data.map(el => {
              if(elI.account_id === el.id){
                this.expertIndustry = elI.payload_value
              }
            })
          })
          // response.data.map(el => {
          //   this.expertIndustry.push(el.expertIndustry)
          //   return el.expertIndustry
          // })
        }else{
          this.expertIndustry = []
        }
      }).catch(error => {
        $('#loading').css({'display': 'none'})
        error
      })
    },
    updateUserStatus(item){
      let parameter = {
        id: item.id.id,
        status: 'BLOCKED'
      }
      $('#loading').css({display: 'block'})
      this.APIRequest('accounts/update_verification', parameter).then(response => {
        $('#loading').css({display: 'none'})
        this.retrieve(null, null)
      })
      $('#profileModal').modal('hide')
    },
    updateStat(item){
      let parameter = {
        id: item.id.id,
        status: 'NOT_VERIFIED'
      }
      $('#loading').css({display: 'block'})
      this.APIRequest('accounts/update_verification', parameter).then(response => {
        $('#loading').css({display: 'none'})
        this.retrieve(null, null)
      })
      $('#profileModal').modal('hide')
    },
    updateStatusByParams(item, status){
      let parameter = {
        id: item.id,
        status: status
      }
      $('#loading').css({display: 'block'})
      this.APIRequest('accounts/update_verification', parameter).then(response => {
        $('#loading').css({display: 'none'})
        this.retrieve(null, null)
      })
      $('#profileModal').modal('hide')
    },
    update(item){
      if(item.status !== 'NOT_VERIFIED'){
        let parameter = {
          id: item.id,
          status: 'GRANTED'
        }
        $('#loading').css({display: 'block'})
        this.APIRequest('accounts/update_verification', parameter).then(response => {
          $('#loading').css({display: 'none'})
          this.retrieve(null, null)
        })
        $('#profileModal').modal('hide')
      }else{
        alert('Not Allowed!')
      }
    }
  }
}
</script>
