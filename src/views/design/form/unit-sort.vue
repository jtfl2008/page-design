<template>
    <div class="form-item col-1">
        <label>{{ config.title }}</label>
        <div>
            <a-select
                v-model="selected"
                style="width:100%"
                size="large"
                @change="handle_change">
                <template v-for="item in options">
                    <a-select-option
                        :key="item.item_id"
                        :value="item.item_id">
                        {{ item.item_title }}
                    </a-select-option>
                </template>
            </a-select>
        </div>
    </div>
</template>

<script>

/**
 * 获取排序列表
 * @param {String} site_code 站点
 * @param {String} pipeline 渠道
 * @param {String} lang 语言
 * @returns {Array} 选项列表
 */
const get_sort_list = ({ site_code, pipeline, lang }) => {
    const url = GESHOP_INTERFACE.gesApi_design_esSearchSortByList.url;
    return new Promise((resolve, reject) => {
        $.ajax({
            url,
            data: {
                site_code,
                pipeline,
                lang
            },
            dataType: 'jsonp',
            success (res) {
                if (res && res.code == 0) {
                    resolve(res.data);
                } else {
                    resolve([]);
                }
            }
        });
    });
};

// Main code
export default {
    name: 'unit-sort',
    props: {
        value: {
            type: String,
            default:''
        },
        // 当前字段的配置
        config: {
            type: Object,
            required: true
        },
        // 根配置
        rootConfig: {
            type: Object,
            required: true
        }
    },

    data () {
        return {
            options: [], // 选项列表
            selected: '', // 选中值
            loading: true // 状态
        }
    },

    methods: {
        /**
         * 切换选项
         * @param {String} value 选中的 item_id
         */
        handle_change (value) {
            this.selected = value;
            this.$emit('input', value);
        }
    },

    async created () {
        // 获取列表
        const page_info = this.$store.state.page.info;
        this.options = await get_sort_list({
            site_code: page_info.site_code,
            pipeline: page_info.pipeline,
            lang: page_info.lang
        });

        // 取消loading状态
        this.loading = false;

        // 先选中当前的值
        if (this.value != '') {
            this.selected = this.value;
        } else {
            // 如果没有保存值，则取第0个值
            if (this.options.length > 0) {
                const item_id = this.options[0].item_id;
                this.handle_change(item_id);
            }
        }
    }
}
</script>