<template>
  <emailScroll ref="scroll"
               :cancel-success="cancelStar"
               :star-success="addStar"
               :getEmailList="getEmailList"
               :emailDelete="emailDelete"
               :star-add="starAdd"
               :star-cancel="starCancel"
               :time-sort="params.timeSort"
               :email-read="emailRead"
               :show-unread="true"
               actionLeft="4px"
               @jump="jumpContent"
  >
    <template #first>
      <Icon class="icon" @click="changeTimeSort" icon="material-symbols-light:timer-arrow-down-outline"
            v-if="params.timeSort === 0" width="28" height="28"/>
      <Icon class="icon" @click="changeTimeSort" icon="material-symbols-light:timer-arrow-up-outline" v-else
            width="28" height="28"/>
      <Icon v-perm="'email:delete'" class="icon" @click="openBatchDelete"
            icon="material-symbols-light:filter-list-off-rounded" width="26" height="26"
            :title="t('batchDeleteEmail')"/>
    </template>
  </emailScroll>

  <el-dialog v-model="batchDeleteShow" :title="t('batchDeleteEmail')" width="380px">
    <div class="batch-delete-form">
      <el-select v-model="batchDeleteField" style="width: 100%">
        <el-option value="sendEmail" :label="t('batchDeleteEmailFieldSender')"/>
        <el-option value="name"      :label="t('batchDeleteEmailFieldName')"/>
        <el-option value="subject"   :label="t('batchDeleteEmailFieldSubject')"/>
      </el-select>
      <el-input
          v-model="batchDeleteKeyword"
          :placeholder="t('batchDeleteEmailKeyword')"
          clearable
          @keyup.enter="confirmBatchDelete"
      />
      <el-button type="danger" :loading="batchDeleteLoading" style="width:100%" @click="confirmBatchDelete">
        {{ t('batchDeleteEmail') }}
      </el-button>
    </div>
  </el-dialog>
</template>

<script setup>
import {useAccountStore} from "@/store/account.js";
import {useEmailStore} from "@/store/email.js";
import {useSettingStore} from "@/store/setting.js";
import emailScroll from "@/components/email-scroll/index.vue"
import {emailList, emailDelete, emailLatest, emailRead, emailBatchDelete} from "@/request/email.js";
import {starAdd, starCancel} from "@/request/star.js";
import {defineOptions, h, onMounted, reactive, ref, watch} from "vue";
import {sleep} from "@/utils/time-utils.js";
import router from "@/router/index.js";
import {Icon} from "@iconify/vue";
import { useRoute } from 'vue-router'
import { useI18n } from 'vue-i18n'

defineOptions({
  name: 'email'
})

const { t } = useI18n()
const route = useRoute();
const emailStore = useEmailStore();
const accountStore = useAccountStore();
const settingStore = useSettingStore();
const scroll = ref({})
const params = reactive({
  timeSort: 0,
})

// 批量删除状态
const batchDeleteShow = ref(false)
const batchDeleteField = ref('sendEmail')
const batchDeleteKeyword = ref('')
const batchDeleteLoading = ref(false)

onMounted(() => {
  emailStore.emailScroll = scroll;
  latest()
})

watch(() => accountStore.currentAccountId, () => {
  scroll.value.refreshList();
})

function changeTimeSort() {
  params.timeSort = params.timeSort ? 0 : 1
  scroll.value.refreshList();
}

function openBatchDelete() {
  batchDeleteKeyword.value = ''
  batchDeleteShow.value = true
}

async function confirmBatchDelete() {
  const kw = batchDeleteKeyword.value.trim()
  if (!kw) {
    ElMessage({ message: t('batchDeleteKeywordEmpty'), type: 'warning', plain: true })
    return
  }
  try {
    batchDeleteLoading.value = true
    const { count } = await emailBatchDelete(kw, batchDeleteField.value)
    if (count === 0) {
      ElMessage({ message: t('batchDeleteEmailEmpty'), type: 'warning', plain: true })
    } else {
      ElMessage({ message: t('batchDeleteEmailSuccess', { count }), type: 'success', plain: true })
      batchDeleteShow.value = false
      batchDeleteKeyword.value = ''
      scroll.value.refreshList()
    }
  } catch(e) {
    console.error(e)
  } finally {
    batchDeleteLoading.value = false
  }
}

function jumpContent(email) {
  emailStore.contentData.email = email
  emailStore.contentData.delType = 'logic'
  emailStore.contentData.showUnread = true
  emailStore.contentData.showStar = true
  emailStore.contentData.showReply = true
  router.push('/message')
}

const existIds = new Set();

async function latest() {
  while (true) {

    let autoRefresh = settingStore.settings.autoRefresh;
    await sleep(autoRefresh > 1 ? autoRefresh * 1000 : 3000);

    if (route.name !== 'email') {
      continue;
    }

    const latestId = scroll.value.latestEmail?.emailId

    if (!scroll.value.firstLoad && autoRefresh > 1) {
      try {
        const accountId = accountStore.currentAccountId
        const allReceive = scroll.value.latestEmail?.allReceive
        const curTimeSort = params.timeSort
        let list = []

        if (accountId === scroll.value.latestEmail?.reqAccountId) {
          list = await emailLatest(latestId, accountId, allReceive);
        }

        if (accountId === accountStore.currentAccountId && params.timeSort === curTimeSort && allReceive === accountStore.currentAccount.allReceive) {
          if (list.length > 0) {

            for (let email of list) {

              email.reqAccountId = accountId;
              email.allReceive = allReceive;

              if (!existIds.has(email.emailId)) {

                existIds.add(email.emailId)
                scroll.value.addItem(email)

                await sleep(50)
              }

            }

          }

        }
      } catch (e) {
        if (e.code === 401 || e.code === 403) {
          settingStore.settings.autoRefresh = 0;
        }
        console.error(e)
      }
    }
  }
}

function addStar(email) {
  emailStore.starScroll?.addItem(email)
}

function cancelStar(email) {
  emailStore.starScroll?.deleteEmail([email.emailId])
}

function getEmailList(emailId, size) {
  const accountId =  accountStore.currentAccountId;
  const allReceive = accountStore.currentAccount.allReceive;
  return emailList(accountId, allReceive, emailId, params.timeSort, size, 0).then(data => {
    data.latestEmail.reqAccountId = accountId;
    data.latestEmail.allReceive = allReceive;
    return data;
  })
}

</script>
<style scoped>
.icon {
  cursor: pointer;
}
.batch-delete-form {
  display: flex;
  flex-direction: column;
  gap: 14px;
  padding-bottom: 6px;
}
</style>
