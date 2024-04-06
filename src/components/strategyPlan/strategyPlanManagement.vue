<script setup>
import axios from "axios";
import moment from "moment";
import { ref, reactive } from "vue";
import Multiselect from "@vueform/multiselect";
import VueDatePicker from "@vuepic/vue-datepicker";
import { useLoaddingStore } from "./../../store/LoaddingStore";
import useBTS from "@/composables/useBTS";
import { getlistPolicy } from "./../../store/getlistPolicy";
import { notify } from "@kyvg/vue3-notification";
import { getBranch } from "../../store/getBranch";
const getListBranch = getBranch();
const itemEdit = ref({});
const originItem = ref({});
const listfillPolicy = getlistPolicy();
const { exportSearchResult } = useBTS(ref);
console.log(listfillPolicy.listgetPolicy);
const loaddingStore = useLoaddingStore();
const listBranch = getBranch();
const year = ref(new Date().getFullYear());
const searchData = reactive({
  sale_policy_id: "",
  br_code: "",
  plan_date: "",
  target_level: 1,
  current_page: 1,
});
const totalRecord = ref(0);
const fromDate = ref(new Date());
const toDate = ref(new Date());
const listBts = ref([]);
const checkListBts = ref(true);
function myCallback(newPage) {
  searchBTS(newPage);
}
const paginate = reactive({
  total_page: "",
  current_page: "",
  total_record: "",
});
const searchBTS = async () => {
  searchData.plan_date = year;
  listBts.value = [];
  getBTSDetail.value = [];
  checkListBts.value = true;
  try {
    loaddingStore.loading = true;
    const response = await axios.post("/api/plan-sale/search", {
      br_code: searchData.br_code,
      plan_date: searchData.plan_date,
      target_level: 1,
      current_page: searchData.current_page,
      paging: true,
    });
    if (
      response &&
      response.data.saleResulResponse &&
      response.data.saleResulResponse.saleResultList.length
    ) {
      listBts.value = response.data.saleResulResponse.saleResultList;
      totalRecord.value = response.data.data.total_record;
      paginate.total_page = response.data.total_page;
      paginate.total_record = response.data.total_record;
      console.log(listBts);
    } else {
      checkListBts.value = false;
    }
    loaddingStore.loading = false;
  } catch (error) {
    loaddingStore.loading = false;
    notify({
      type: "error",
      title: "Get current user",
      text: error,
      duration: 500,
    });
  }
};
const getBTSDetail = ref({
  branch_id: 0,
  bts_code: 0,
  start_date: 0,
  end_date: 0,
  sale_policy_id: 0,
  status: 0,
});
const showSaveButton = ref(false);
const checkDataDetail = ref(false);
const getDetailEdit = async (item, data) => {
  itemEdit.value = [];
  itemEdit.value = data;
  originItem.value = { ...data };
  showSaveButton.value = false;
  if (item) {
    getBTSDetail.value = JSON.parse(JSON.stringify(item));
  } else {
    checkDataDetail.value = true;
  }
  console.log(item);
};
const dateTime = (date) => {
  var getDate = ref("");
  if (date) {
    getDate.value = moment(date).format("DD-MM-YYYY");
    return getDate.value;
  } else {
    return getDate.value;
  }
};
function clearListBTS() {
  searchBTS();
}
function exportSearchResultBTS() {
  searchData.from_date = fromDate.value
    ? moment(fromDate.value).format("DD/MM/yyyy")
    : "";
  searchData.to_date = toDate.value
    ? moment(toDate.value).format("DD/MM/yyyy")
    : "";
  exportSearchResult({ ...searchData });
}
const getQuestionDelete = async (item) => {
  console.log(item.id);
  if (item) {
    getBTSDetail.value.id = item.id;
    getBTSDetail.value.status = item.status;
  } else {
    checkDataDetail.value = true;
  }
  console.log(getBTSDetail.value);
};
function selectBoxBranch(selectedValue) {
  console.log(selectedValue);
  searchData.br_code = "";
  const selectedOption = getListBranch.getBranch.find(
    (option) => option.value === selectedValue
  );
  if (selectedOption) {
    searchData.br_code = selectedOption.code;
  }
  console.log(searchData.br_code);
}
</script>

<template>
  <div class="card-body p-0">
    <div class="row mb-3">
      <div class="col align-self-start">
        <div class="mb-3 row">
          <label class="col-sm-4 col-form-label"
            >{{ $t("HOME.SEARCH.COMBOBOX.BRANCHE") }}:</label
          >
          <div class="col-sm-8">
            <Multiselect
              v-model="br_code"
              :options="listBranch.getBranch || []"
              track-by="code"
              label="label"
              @input="selectBoxBranch"
              :placeholder="$t('SELECT_ONE')"
            ></Multiselect>
          </div>
        </div>
      </div>
      <div class="col align-self-start">
        <div class="mb-3 row"></div>
      </div>
      <div class="col align-self-start">
        <div class="mb-3 row"></div>
      </div>
    </div>

    <div class="row mb-3">
      <div class="col align-self-start">
        <div class="mb-3 row">
          <label class="col-sm-4 col-form-label">{{ $t("FROM_DATE") }}:</label>
          <div class="col-sm-8">
            <VueDatePicker
              v-model="year"
              auto-apply
              year-picker
              format="yyyy"
            ></VueDatePicker>
          </div>
        </div>
      </div>
      <div class="col align-self-start">
        <div class="mb-3 row"></div>
      </div>

      <div class="col align-self-start">
        <div class="mb-3 row"></div>
      </div>
    </div>

    <div class="row mb-3">
      <div class="btn-showcase text-center">
        <button
          class="btn btn-pill btn-primary-gradien"
          type="button"
          @click="searchBTS"
        >
          {{ $t("SEARCH") }}
        </button>
        <button
          class="btn btn-pill btn-add"
          type="button"
          @click="exportSearchResultBTS"
        >
          {{ $t("EXPORT_DETAIL") }}
        </button>
      </div>
    </div>

    <div class="row mb-3 evaluations">
      <fieldset v-if="!checkListBts" class="table-responsive">
        <div class="text-center no-item">{{ $t("NO.ITEM") }}</div>
      </fieldset>
      <fieldset v-if="listBts.length" class="table-responsive">
        <div class="table-responsive">
          <table class="table table-bordered table-hover">
            <thead
              class="table-success"
              style="background-color: #24695c; color: #fff"
            >
              <tr>
                <th scope="col">#</th>
                <th scope="col">{{ $t("HOME.SEARCH.COMBOBOX.BRANCHE") }}</th>
                <th scope="col">{{ $t("BTS_CODE") }}</th>
                <th scope="col">{{ $t("STATUS") }}</th>
                <th scope="col">{{ $t("POLICY") }}</th>
                <th scope="col">{{ $t("FROM_DATE") }}</th>
                <th scope="col">{{ $t("TO_DATE") }}</th>
                <th scope="col">{{ $t("HOME.SEARCH.MAP.ACTION") }}</th>
              </tr>
            </thead>
            <tbody v-for="(item, index) in listBts" :key="item">
              <tr>
                <td>{{ (searchData.current_page - 1) * 10 + (index + 1) }}</td>
                <td>{{ item.br_code }}</td>
                <td>{{ item.bts_code }}</td>
                <td>{{ item.status == 1 ? "Active" : "Inactive" }}</td>
                <td>{{ item.sale_policy_name }}</td>
                <td>{{ dateTime(item.start_date) }}</td>
                <td>{{ dateTime(item.end_date) }}</td>
                <td v-if="item.sale_policy_id != '5'">
                  <div class="buttons">
                    <button
                      class="btn btn-primary btn-xs"
                      data-bs-toggle="modal"
                      data-original-title="test"
                      data-bs-target="#exampleModalEditBTS"
                      @click="getDetailEdit(item, data)"
                    >
                      <i class="fa fa-edit"></i>
                      <!-- {{ $t('EDIT') }} -->
                    </button>
                    <button
                      class="btn btn-primary btn-xs"
                      data-bs-toggle="modal"
                      data-original-title="test"
                      data-bs-target="#exampleModalConfirmActive"
                      @click="getQuestionDelete(item)"
                      :class="item.status === 1 ? 'btn-danger' : 'btn-primary'"
                      style="
                        margin-left: 10px;
                        width: 100px;
                        height: 26.5px;
                        font-size: 14px;
                      "
                    >
                      {{ item.status === 1 ? "Inactive" : "Active" }}
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
          <div class="pagination-search">
            <pagination
              v-model="searchData.current_page"
              :records="totalRecord"
              :per-page="10"
              @paginate="myCallback"
            />
          </div>
        </div>
      </fieldset>
    </div>
    <btsEdit
      @clearValue="clearListBTS"
      :itemAdd="getBTSDetail"
      v-model="getBTSDetail"
      :item="item"
      :options="options"
      :listPolicy="listPolicy"
      :ListSaleTime="ListSaleTime"
      :start_date="getBTSDetail.start_date"
      :end_date="getBTSDetail.end_date"
    />
    <confirmEditActive
      @clearValue="clearListBTS"
      :itemAdd="getBTSDetail"
      v-model="getBTSDetail"
      :item="item"
    />
  </div>
</template>

<style lang="scss" scoped>
.multiselect {
  max-height: 35px;
  min-height: 30px;
}
.dp__input_wrap {
  height: 35px;
  display: flex;
}
.evaluations {
  fieldset {
    padding: 15px;
    margin: 0 2px;
    border: 1px solid #c0c0c0;
  }
  legend {
    float: initial;
    font-size: 14px;
    width: auto;
    border-bottom: none;
    padding: 0px 5px;
  }
  .no-item {
    font-weight: 500;
  }
  svg {
    width: 16px;
    height: 16px;
  }
}
</style>
