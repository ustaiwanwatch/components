<template>
  <div :class="{ phone: isPhone }" class="overview-card">
    <h1 class="overview-card-title">{{ $t('BillOverviewCard.labelTitle') }}</h1>
    <div class="bill-meta">
      <span class="bill-meta-info">{{ bill.billCode }}</span>
      <span class="bill-meta-info">{{ bill.billType.code | billType }}</span>
    </div>
    <h1 class="bill-title">{{ bill.title }}</h1>
    <Row>
      <Col :span="24" class="overview-card-info-block">
      <!-- Sponsor -->
      <span class="label">{{ $t('BillOverviewCard.labelSponsor') }}</span>
      <div class="bill-sponsor">
        <img :class="avatarClass" :src="avatarSource" :style="avatarStyle" class="avatar">
        <router-link v-if="bill.sponsor" :to="`/members/${bill.sponsor.id}`">
          <p class="name">{{ bill.sponsor.role.title }} {{ bill.sponsor.firstName }} {{ bill.sponsor.lastName }}</p>
        </router-link>
        <p class="area">{{ memberArea }}</p>
      </div>
      </Col>
      <Col :span="isDesktop ? 8 : 12" class="overview-card-info-block">
      <!-- Congress -->
      <span class="label">{{ $t('BillOverviewCard.labelCongress') }}</span>
      <p class="value">{{ bill.congress }}th</p>
      </Col>
      <Col :span="isDesktop ? 8 : 12" class="overview-card-info-block">
      <!-- Introduced Date -->
      <span class="label">{{ $t('BillOverviewCard.labelIntroducedDate') }}</span>
      <p class="value">{{ bill.introducedDate | localTime }}</p>
      </Col>
      <Col :span="isDesktop ? 8 : 12" class="overview-card-info-block">
      <!-- Cosponsors -->
      <span class="label">{{ $t('BillOverviewCard.labelCosponsors') }}</span>
      <p class="value">{{ bill.cosponsors ? bill.cosponsors.length : 0 }}</p>
      </Col>
      <Col :span="24" class="overview-card-info-block">
      <!-- Tracker -->
      <span class="label">{{ $t('BillOverviewCard.labelStatus') }}</span>
      <p class="value">{{ billLatestAction | trimConGovAction }}</p>
      <BillTracker :steps="bill.trackers" :progress="billProgress" class="bill-tracker"/>
      </Col>
    </Row>
  </div>
</template>

<script>
import BillTracker from '~/components/BillTracker'
import defaultAvatar from '~/assets/img/tw-logo-color.png'

export default {
  components: {
    BillTracker
  },
  props: {
    bill: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      avatarSize: 40
    }
  },
  computed: {
    isDesktop () {
      return this.$store.getters.isDesktop
    },
    isPhone () {
      return this.$store.getters.isPhone
    },
    style () {
      return `
        object-fit: cover;
        width: ${this.size}px;
        height: ${this.size}px;
      `
    },
    avatarSource () {
      const pictures = this.bill.sponsor && this.bill.sponsor.profilePictures
      return pictures && pictures.tiny ? pictures.tiny : defaultAvatar
    },
    avatarStyle () {
      return `
        object-fit: cover;
        width: ${this.avatarSize}px;
        height: ${this.avatarSize}px;
      `
    },
    avatarClass () {
      let color = ''
      if (!this.bill.sponsor || !this.bill.sponsor.role) {
        return 'gray'
      }
      switch (this.bill.sponsor.role.party) {
        case 'Republican':
          color = 'red'
          break
        case 'Democrat':
          color = 'blue'
          break
        default:
          color = 'gray'
          break
      }
      return color
    },
    memberArea () {
      if (!this.bill.sponsor || !this.bill.sponsor.role) {
        return ''
      }

      if (this.bill.sponsor.role.district) {
        return `, ${this.bill.sponsor.role.state}-${this.bill.sponsor.role.district}`
      } else {
        return `, ${this.bill.sponsor.role.state}`
      }
    },
    billProgress () {
      if (_.isEmpty(this.bill.trackers)) {
        return 0
      }

      const totalSteps = this.bill.trackers.length
      let currentStep
      this.bill.trackers.forEach((step, index) => {
        if (step.selected) currentStep = index + 1
      })
      return (currentStep / totalSteps) * 100
    },
    billLatestAction (a, b, c) {
      let latestActionTime = 0
      let latestAction = ''
      if (!_.isEmpty(this.bill.actions)) {
        this.bill.actions.forEach(action => {
          if (action.datetime > latestActionTime) {
            latestAction = action.description
            latestActionTime = action.datetime
          }
        })
      }
      if (process.browser) {
        // strip html tags from the string
        var dom = document.createElement('DIV')
        dom.innerHTML = latestAction
        latestAction = dom.textContent || dom.innerText || ''
      }
      return latestAction
    }
  }
}
</script>

<style scoped lang="scss">
@import 'assets/css/app';
@import 'assets/css/typography';
@import 'assets/css/colors';

.overview-card {
  @extend .card;
  margin-bottom: 20px;

  .overview-card-title {
    @extend .card-title;
  }
}

.overview-card-info-block {
  @extend .card-info-block;

  .categories {
    display: flex;
    flex-wrap: wrap;

    .category {
      margin-right: 10px;
      background: $twGrayLighter;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 5px;

      &:last-child {
        margin-right: 0px;
      }

      img {
        height: 14px;
      }
    }
  }
}

.bill-meta {
  display: flex;
  align-items: center;
  margin-bottom: 10px;

  .bill-meta-info {
    font-size: 1em;
    font-weight: $twSemiBold;
    color: $twGrayLight;
    background: $twGrayLighter;
    border-radius: 10px;
    padding: 1px 8px;
    margin-right: 5px;

    &.success {
      color: $twWhite;
      background: $twGreen;
    }
  }
}

.bill-title {
  font-size: 1.4em;
  font-weight: $twSemiBold;
  color: $twGrayDark;
  margin-bottom: 20px;
}

.bill-sponsor {
  display: flex;
  align-items: center;
  margin-top: 10px;

  .avatar {
    border-radius: 50%;
    border-style: solid;
    border-width: 3px;
    margin-right: 10px;

    &.red {
      border-color: $twRed;
    }

    &.blue {
      border-color: $twBlue;
    }

    &.gray {
      border-color: $twGrayLight;
    }
  }

  .name {
    font-size: 1em;
    color: $twGrayDark;
    // font-weight: $twSemiBold;
    &:hover {
      color: $twIndigo;
    }
  }

  .area {
    font-size: 1em;
    color: $twGrayDark;
    // font-weight: $twSemiBold;
  }
}

.bill-tracker {
  margin-top: 20px;
}
</style>

<style lang="scss">
@import 'assets/css/app';
@import 'assets/css/typography';
@import 'assets/css/colors';

.ivu-tooltip-rel {
  display: inherit;
}
</style>
