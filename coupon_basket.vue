<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header">
                    <div class="main_container position_relative">
                        <h2>My Basket</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div class="row margin_40">
        		        <div class="col-md-6 clearfix">
        		            <router-link to="/coupons">
        		                <div class="animated_btn basket_btn">
        		                    Back to Coupons
        		                </div>    
        		            </router-link>
        		        </div>
        		        <div class="col-md-6 clearfix">
    		                <div @click="printPage()" class="animated_btn coupon_btn">
    		                    Print Coupons
    		                </div>
        		        </div>
        		    </div>
                    <div class="row">
                        <div v-for="(item, index) in couponsInBasket" :key="item.id" class="col-md-6 col-sm-6 col-xs-12"> 
                         <!--v-on:remove="couponList.splice(index, 1)"-->
                            <div :id="item.id" class="row coupon_container">
                                <div class="col-md-6 col-sm-6 col-xs-12">
                                    <div class="coupon_img">
                                        <img class="img_max" :src="item.store_logo" :alt="item.store.name + ' Logo'" />
                                    </div>
                                </div>	
                                <div class="col-md-6 col-sm-6 col-xs-12">
                                    <div class="coupon_content">
                                        <i v-on:click="removeCoupon(item)" class="fas fa-times"></i>
                                        <div>
                                            <p v-if="item.store">{{ item.store.name }}</p>
                                        	<h4>{{ item.name_short }}</h4>
                                        	<p class="coupon_dates"><span v-if="isMultiDay(item)">{{ item.start_date | moment("MM/DD/YYY", timezone)}} - {{ item.end_date | moment("MM/DD/YYY", timezone)}}</span><span v-else>{{ item.start_date | moment("MM/DD/YYY", timezone)}}</span></p>
                                            <router-link :to="{ name: 'couponDetails', params: { id: item.slug }}">
                                                <p class="event_link">Coupon Details <i class="fas fa-angle-double-right"></i></p>
                                            </router-link>
                                        </div>
                                    </div>
                                </div>
                            </div> 
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
              
<script>
    define(["Vue", "vuex", "jquery", 'js-cookie', "moment", "moment-timezone", "vue-moment", "vue-lazy-load"], function (Vue, Vuex, $, Cookies, moment, tz, VueMoment, VueLazyload) {
        Vue.use(VueLazyload);
        return Vue.component("coupon-basket-component", {
            template: template, // the variable template will be injected,
            props: ['selected'],
            data: function () {
                return {
                    dataLoaded: false,
                    selectedCoupons: null,
                    events: [],
                    moreEvents: [],
                    moreEventsFetched: false,
                    noMoreEvents: false,
                    noEvents: false,
                    couponsInBasket:null
                }
            },
            created (){
                this.loadData().then(response => {
                    this.selectedCoupons = Cookies.get('coupon_ids');
                    this.selectedCoupons = JSON.parse(this.selectedCoupons);
                    this.dataLoaded = true;
                    this.couponsInBasket = this.couponList;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findCouponById'
                ]),
                couponList: function couponsList() {
                    if (this.selectedCoupons) {
                        var vm = this;
                        var temp_coupon = [];
                        _.forEach(this.selectedCoupons, function(value, key) {
                            var current_coupon = vm.findCouponById(value);
                            if (current_coupon.store !==null && current_coupon.store !==undefined && _.includes(current_coupon.store.store_front_url_abs, 'missing')) {
                                current_coupon.store_logo = vm.property.default_logo
                            } else {
                                current_coupon.store_logo = current_coupon.store.store_front_url_abs;
                            }
                                
                            current_coupon.name_short = _.truncate(current_coupon.name, { 'length': 30, 'separator': ' ' });
                            
                            temp_coupon.push(current_coupon);
                        }); 
    
                        var sortedCoupons = _.orderBy(temp_coupon, [function(o) { return o.end_date; }]);
                        
                        return sortedCoupons;    
                    }
                }
            },
            methods: {
                loadData: async function () {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "coupons")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                isMultiDay(item) {
                    var timezone = this.timezone
                    var start_date = moment(item.start_date).tz(timezone).format("MM-DD-YYYY")
                    var end_date = moment(item.end_date).tz(timezone).format("MM-DD-YYYY")
                    if (start_date === end_date) {
                        return false
                    } else {
                        return true
                    }
                },
                selectedCoupon(item) {
                    console.log(item)    
                },
                removeCoupon(item){
                    var vm = this;
                    var new_list = _.filter(vm.couponsInBasket,function(o){return o.id !==item.id;})
                    console.log("new_list", new_list);
                    vm.couponsInBasket = new_list;
                    //update the cache
                    // Add updated coupon list to localstorage
                    Cookies.set('coupon_ids', '');
                    Cookies.set('coupon_ids', JSON.stringify(vm.selected_coupon_id));
                }
            }
        });
    });
</script>