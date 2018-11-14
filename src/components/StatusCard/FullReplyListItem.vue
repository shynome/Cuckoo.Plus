<template>
  <div class="full-reply-list-item" v-loading="isListItemLoading" @mouseover="onItemMouseOver" @mouseout="onItemMouseOut">
    <div class="left-area">
      <mu-avatar class="status-replier-avatar" slot="avatar" size="34">
        <img :src="status.account.avatar_static">
      </mu-avatar>
    </div>
    <div class="center-area">

      <div class="reply-user-display-name">
        <p class="primary-read-text-color">
          {{getAccountDisplayName(status.account)}}
          <span class="at-name secondary-read-text-color">@{{getAccountAtName(status.account)}}</span>
        </p>
        <span v-if="status.favourites_count > 0"
              class="reply-favorites-count"
              :class="[ status.favourited ? 'primary-theme-text-color' : 'secondary-read-text-color' ]">
                  +{{status.favourites_count}}
                </span>
      </div>

      <mu-card-text class="status-content full-reply-status-content" v-html="formatHtml(status.content)"></mu-card-text>

      <div class="full-reply-attachment-area">
        <media-panel :mediaList="status.media_attachments" :pixivCards="status.pixiv_cards" :sensitive="status.sensitive"/>
      </div>

      <div class="reply-action-list">

        <a class="reply-button secondary-theme-text-color"
           @click="onReplyToStatus()">{{$t($i18nTags.statusCard.reply_to_replier)}}</a>

        <div class="plus-one-button secondary-theme-text-color"
             @click="onFavoriteButtonClick()"
             :class="{ 'primary-theme-bg-color': status.favourited }">
          <a>+1</a>
        </div>

      </div>

    </div>
    <div class="right-area">
      <span v-show="!shouldOpenMoreOperationPopOver && !shouldShowMoreOperationTriggerBtn" class="reply-from-now secondary-read-text-color">{{getFromNowTime()}}</span>
      <mu-button v-show="shouldOpenMoreOperationPopOver || shouldShowMoreOperationTriggerBtn" icon style="width: 16px; height: 16px" @click="onOpenMoreOperationPopOver">
        <mu-icon class="header-icon secondary-read-text-color" value="more_vert"/>
      </mu-button>
    </div>

    <mu-popover cover placement="left-start"
                :open.sync="shouldOpenMoreOperationPopOver"
                :trigger="moreOperationTriggerBtn">
      <mu-list>
        <mu-list-item button v-if="currentUserAccount.id === status.account.id"
                      @click="onDeleteStatus()">
          <mu-list-item-title>Delete</mu-list-item-title>
        </mu-list-item>
      </mu-list>
    </mu-popover>
  </div>
</template>

<script lang="ts">
  import { Vue, Component, Prop } from 'vue-property-decorator'
  import { Getter, Action, State } from 'vuex-class'
  import { formatHtml, getVisibilityDescInfo } from '@/util'
  import * as moment from 'moment'
  import { mastodonentities } from "@/interface"
  import MediaPanel from './MediaPanel'

  @Component({
    components: {
      'media-panel': MediaPanel
    }
  })
  class FullReplyListItem extends Vue {

    $confirm

    @Prop() status: mastodonentities.Status

    @State('currentUserAccount') currentUserAccount: mastodonentities.AuthenticatedAccount

    @Action('updateFavouriteStatusById') updateFavouriteStatusById
    @Action('deleteStatus') deleteStatus

    @Getter('getAccountDisplayName') getAccountDisplayName
    @Getter('getAccountAtName') getAccountAtName

    formatHtml = formatHtml

    isListItemLoading: boolean = false

    shouldShowMoreOperationTriggerBtn: boolean = false
    shouldOpenMoreOperationPopOver: boolean = false
    moreOperationTriggerBtn = null


    getFromNowTime () {
      return moment(this.status.created_at).fromNow(true)
    }

    onFavoriteButtonClick () {
      this.updateFavouriteStatusById({
        favourited: !this.status.favourited,
        targetStatusId: this.status.id
      })
    }

    onReplyToStatus () {
      this.$emit('reply', this.status)
    }

    onOpenMoreOperationPopOver (e) {
      this.moreOperationTriggerBtn = e.target
      this.shouldOpenMoreOperationPopOver = true
    }

    async onDeleteStatus () {
      const doDeleteStatus = (await this.$confirm('要删除这条嘟文吗?', '', {})).result
      if (doDeleteStatus) {
        this.isListItemLoading = true
        await this.deleteStatus({ statusId: this.status.id })
      }
    }

    onItemMouseOver () {
      this.shouldShowMoreOperationTriggerBtn = true
    }

    onItemMouseOut () {
      this.shouldShowMoreOperationTriggerBtn = false
    }

  }

  export default FullReplyListItem
</script>

<style lang="less" scoped>
  .full-reply-list-item {
    position: relative;
    display: flex;
    padding: 12px 16px;

    .center-area {
      flex-grow: 1;
      margin: 0 10px 0 16px;
      display: flex;
      flex-direction: column;

      .reply-user-display-name {
        display: flex;
        align-items: center;

        > p {
          margin: 0;
          font-size: 15px;
          font-weight: 500;
          text-overflow: ellipsis;
          overflow: hidden;
        }

        .reply-favorites-count {
          font-size: 13px;
          font-weight: 500;
          margin-left: 8px;
        }
      }

      .full-reply-status-content {
        padding: 0;
      }

      .reply-action-list {
        display: flex;
        align-items: center;
        margin-top: 6px;

        .common-style-mixin() {
          cursor: pointer;
          font-size: 13px;
          margin: 0 8px;
        }

        .reply-button {
          .common-style-mixin();
          margin-left: 0;
        }

        .plus-one-button {
          .common-style-mixin();
          width: 24px;
          height: 24px;
          line-height: 24px;
          text-align: center;
          border-radius: 50%;
        }
      }
    }

    .right-area {
      display: flex;
      flex-shrink: 0;
      width: 38px;

      .reply-from-now {
        font-size: 13px;
      }
    }
  }
</style>