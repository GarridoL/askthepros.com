<template>
  <div class="container-fluid mb-5">
		<h2 style="margin-top: 25px;">Checkout Summary</h2>
    <p v-if="errorMessage" class="text-danger">
      <b>Opps!</b> {{errorMessage}}
    </p>
    <roudedBtn
      :onClick="() => {
        this.redirect('/Subscription')
      }"
      v-if="selected === null && isLoading === false"
      :text="'Go back'"
      :styles="{
        backgroundColor: '#01004E',
        color: 'white'
      }"
    />
		<div class="col-sm-7 p-0 mt-3" v-if="selected">
			<table class="table table-striped table-bordered">
				<thead>
					<tr>
						<th scope="col">DESCRIPTION</th>
						<th scope="col">AMOUNT</th>
					</tr>
				</thead>
				<tbody>
					<tr>
						<td>{{selected.category.toUpperCase()}} Monthly Subscription</td>
						<td>$ {{selected.payload_value}} / Month</td>
					</tr>
          <tr>
            <td></td>
            <td>
              <select class="form-control" v-model="months">
                <option v-for="(item, index) in 12" :key="index" :value="item">{{item + ' Month' + (item > 1 ? 's' : '')}}</option>
              </select>
            </td>
          </tr>
					<tr>
						<td><b>Total</b></td>
						<td>$ {{parseInt(selected.payload_value) * months}}</td>
					</tr>
				</tbody>
			</table>
		</div>
    <div class="mt-1 choose_payment">
		  <b>Payment Method</b>
    </div>
    <!-- <div class="d-flex stripe_btn_container mt-3">
      <i id="stripe" class="far fa-dot-circle" v-if="payment" @click="choose"></i>
      <i id="stripe" class="far fa-circle"  v-if="!payment" @click="choose"></i>
      <label for="stripe" class="stripe_label">Stripe</label>
    </div> -->
    <div class="mt-3 icons-fontawesome">
      <i class="fa fa-cc-visa"></i>
      <i class="fa fa-cc-discover"></i>
      <i class="fa fa-cc-mastercard"></i>
      <i class="fa fa-cc-amex"></i>
      <i class="fa fa-cc-jcb"></i>
    </div>
    <div class="col-sm-6 p-0 mt-3" v-if="selected !== null">
      <div class="mt-3 d-flex justify-content-start">
        <stripe-cc ref="stripe" :data="{
          ...selected,
          months: months,
          total: selected ? months * parseInt(selected.payload_value) : 0,
          currency: 'USD',
          title: selected ? selected.category.toUpperCase() + ' Monthly Subscription in AskThePros' : ''
        }"></stripe-cc>
      </div>
      <div class="mt-3 d-flex justify-content-start">
        <div>
          <i class="far fa-square" :style="{
            cursor: 'pointer'
          }" v-if="!isAgree" @click="agree"></i>
          <i class="far fa-check-square" :style="{
            cursor: 'pointer'
          }" v-if="isAgree" @click="agree"></i>
        </div>
        <div class="ml-2">
          <p class="mb-2"><i :style="{
            color: 'red'
          }">*</i>I agree my card will be automatically billed monthly until cancellation.</p>
        </div>
      </div>
      <div style="margin-top: 25px">
        <roudedBtn
          :onClick="checkout"
          :text="'Checkout'"
          :styles="{
            backgroundColor: '#01004E',
            color: 'white'
          }"
        />
      </div>
    </div>
	</div>
</template>

<script>
import roudedBtn from 'src/modules/generic/roundedBtn'
import ROUTER from 'src/router'
export default {
  mounted(){
    this.retrieve()
  },
  data() {
    return {
      payment: true,
      isValid: true,
      isAgree: false,
      subscriptions: null,
      selected: null,
      months: 1,
      errorMessage: null,
      isLoading: false
    }
  },
  components: {
    roudedBtn,
    'stripe-cc': require('modules/payments/Stripe.vue')
  },
  methods: {
    redirect(route){
      ROUTER.push(route)
    },
    choose() {
      this.payment = !this.payment
    },
    agree() {
      this.isAgree = !this.isAgree
    },
    checkout() {
      this.errorMessage = null
      if(this.isAgree){
        this.$refs.stripe.addNewPaymentMethod()
      }else{
        this.errorMessage = 'Please accept the I agree my card will be automatically billed monthly until cancellation.'
      }
    },
    retrieve(){
      console.log('test')
      let plan = this.$route.params.plan
      plan = plan.replace('_', ' ')
      let parameter = {
        condition: [{
          value: 'subscriptions',
          clause: '=',
          column: 'payload'
        }, {
          value: plan,
          clause: '=',
          column: 'category'
        }
        ]
      }
      $('#loading').css({'display': 'block'})
      this.isLoading = true
      this.APIRequest('payloads/retrieve', parameter).then(response => {
        $('#loading').css({'display': 'none'})
        if(response.data.length > 0) {
          this.selected = response.data[0]
        }else{
          this.selected = null
        }
        this.isLoading = false
      }).catch(error => {
        $('#loading').css({'display': 'none'})
        this.isLoading = false
        error
      })
    }
  }
}
</script>

<style lang="scss" scoped>
@import "~assets/style/colors.scss";
.choose_payment {
  margin-top: 30px !important;
}

.container-fluid{
  min-height: 70vh;
}

#stripe {
  color: $darkPrimary !important;
  cursor: pointer;
}
.stripe_btn_container {
  align-items: center;
}
.stripe_label {
  display: inline;
  margin-bottom: 0;
  margin-left: 15px;
  font-weight: 500;
}
.form-control{
  height: 50px !important;
  border-radius: 25px !important;
}

.icons-fontawesome{
  font-size: 32px;
  color: $primary;
}
</style>
