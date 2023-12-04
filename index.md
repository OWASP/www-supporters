---

layout: col-sidebar
title: OWASP Supporter Portal
tags: OWASP supporter
maintenance: false

---

<!-- rebuild 001 -->

{% if page.maintenance %}
The Supporter portal is currently undergoing maintenance and will return soon. Thank you for your patience.
{% else %}
<style>
[v-cloak] {display: none}

#member-qr {
  float:right;
  padding: 16px;
}

.label {
  font-weight: bold;
  margin-right: 8px;
}

.info, .multi-info {
  margin-bottom:16px;
  margin-left: 75px;
}

label {
  font-weight: bold;
  margin-right:8px;
}

button {
  margin-right: 16px;
}

.small {
  padding: 2px 8px;
}

.errors {
  padding-bottom: 24px;
  padding-top: 12px;
  border-top: 3px dotted red;
}
.error {
  font-weight:bold;
  color: darkred;
  border-left: 5px solid red;
  padding-left: 8px;
}

.info-section {
  border: 3px solid darkblue;
  border-radius: 8px;
  padding: 8px;
  margin-top: 40px;
}
.section-label {
  margin-top: -20px;
  background: white;
}

.capitalize {
    text-transform: capitalize;
}

.danger-button {
  background-color: #dc3545;
}
</style>

{% raw %}
  <div id="supporter-portal-app" style="margin: 0px" v-cloak>
    <div id="supporter-qr"></div>
    <div id="errors" v-if="errors.length > 0">
      <label>Please correct the following:</label>
      <template v-for="(err, i) in errors">
        <template v-for="(msg, ii) in err">
          <div class="error">{{ msg }}</div>
        </template>
      </template>
    </div>
    <div
      id="supporter-not-found"
      v-if="!supporter_ready && mode == 0 && !loading && !supporter_logged_out"
    >
      No corporate supporter information was found or your support has expired. Please contact
      <a href="mailto:kelly.santalucia@owasp.com"
        ><button class="cta-button">Corporate Supporter Assistance</button></a
      >
      <br />
      If you feel this message is in error, submit a JIRA ticket at
      <a href="/">Supporter Portal Support</a>
    </div>
    <div
      id="supporter-logged-out"
      v-if="supporter_logged_out && mode == 0 && !loading"
    >
      Your session has expired. Please
      <a href="https://supporters.owasp.org/"
        ><button class="cta-button">Log In</button></a
      >
      <br />
    </div>
    <div id="supporter-info" class="info-section" v-if="supporter_ready && mode == 0">
      <h3 class="section-label">Welcome, {{ supporter_data.name }}</h3>
      <br />
      <section v-if="supporter_data['supporter_number']">
        <div class="label">Supporter Number:</div>
        <div class="info">
          {{
              supporter_data.supporter_number.substring(
              supporter_data.supporter_number.lastIndexOf("/") + 1
            )
          }}
        </div>
      </section>
      <section v-else>
        <div class="label">Supporter Number:</div>
        <div class="info">
          Data not found. Contact
          <a href="mailto:kelly.santalucia@owasp.com">Corporate Supporter Assistance</a>
        </div>
      </section>
      <div class="label">Supporter Type:</div>
      <section id="membership" v-if="supporter_data.supporter_type">
        <div class="info">{{ supporter_data.supporter_type }}</div>
        <div class="label">Supporter Membership End:</div>
        <div class="info">{{ supporter_data.supporter_end }}</div>        
      </section>      
    </div>
    <div id="loading" v-if="loading">
      This may take a few moments...
      <button class="cta-button" style="width: 80px; height: 80px">
        <div class="spinner">
          <div class="inner-spinner"></div>
        </div>
      </button>
    </div>
  </div>
{% endraw %}

<script src="https://js.stripe.com/v3"></script>
<script src="https://unpkg.com/vue@2"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.15/lodash.min.js"></script>
<script>
var stripe = Stripe('pk_live_mw0B2kiXQTFkD44liAEI03oT00S5AGfSV3');
window.addEventListener('load', function() {
  new Vue({
    el: '#supporter-portal-app',
    data() {
      return {
        loading: true,
        errors: [],
        supporter_data: null,
        update_interval: null,
        mode: 0,
        saved_data: null,
        supporter_logged_out: false,
        pendingCancellation: false,        
      };
    },
    created: function () {
      if (this.loading) {
        this.getSupporterInfo();        
      } // end if loading
    },
    computed: {
      supporter_ready() {
        return (
          !this.loading &&
          this.supporter_data != null &&
          this.supporter_data.name
        );
      }
    },
    methods: {
      getSupporterInfo: function () {
        const postData = {
            params: {
              authtoken: Cookies.get("CF_Authorization"),
            },
          };
      // for now assuming this is local testing
      this.supporter_data = {}
      this.supporter_data.supporter_type = 'Gold'
      this.supporter_data['name'] = 'Harold Test Data'
      this.supporter_data['supporter_end'] = '2022-02-11'
      this.supporter_data.membership_email = 'harold.blankenship@owasp.com'
      this.supporter_data['supporter_number'] = 'cst_34249829348298439283749'
      this.supporter_data['supporter-qr'] = 'https://owasp.org'
      this.supporter_logged_out = false
      this.loading=false
      // setTimeout(function(membership_data) { 
      //       if(membership_data && membership_data['name']) {
      //           el = kjua({text: membership_data['member_number']});
      //           div = document.getElementById('member-qr');
      //           if(div) {
      //             div.appendChild(el)
      //           }
      //       }
      //   }, 1000, this.membership_data)
      this.saved_data = JSON.parse(JSON.stringify(this.supporter_data))
            //  */
      this.$forceUpdate();
      },
      validate: function () {
        this.errors = [];
        return this.errors.length === 0;
      },
      switchMode: function () {
        this.mode = !this.mode;
        if (this.saved_data) {
          this.supporter_data = JSON.parse(JSON.stringify(this.saved_data));
        }
        this.errors = []; // why doesn't this set errors to empty?
        this.$forceUpdate();
        return false;
      }
    }, // end methods
  }) // end Vue
}, false) // end addEventListener
</script>
{% endif %}
