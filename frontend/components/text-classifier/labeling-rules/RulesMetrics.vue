<template>
  <div class="rule-metrics__container">
    <p class="rule-metrics__title">{{ title }}</p>
    <div class="rule-metrics">
      <template>
        <div
          class="rule-metrics__item"
          v-for="metric in metrics"
          :key="metric.name"
        >
          <p class="metric__title" :data-title="metricsTitle(metric)">
            {{ metric.name }}
          </p>
          <p class="metric__rule">
            <transition name="fade" mode="out-in" appear>
              <strong :key="metric.rule.value">{{ metric.rule.value }}</strong>
            </transition>
          </p>
          <span class="metric__records" v-if="metric.records">
            <transition name="fade" mode="out-in" appear>
              <span
                v-html="`${metric.records.value}/${metric.records.total}`"
                :key="metric.records.value"
              />
            </transition>
          </span>
        </div>
      </template>
    </div>
    <slot name="button-bottom" />
  </div>
</template>

<script>
import { TextClassificationDataset } from "@/models/TextClassification";

export default {
  props: {
    dataset: {
      type: TextClassificationDataset,
      required: true,
    },
    title: {
      type: String,
      required: true,
    },
  },
  data: () => {
    return {
      metricsByRules: {},
    };
  },
  computed: {
    rules() {
      return this.dataset.labelingRules;
    },
    metricsTotal() {
      return this.dataset.labelingRulesOveralMetrics || {};
    },

    currentRule() {
      return this.dataset.getCurrentLabelingRule();
    },

    ruleMetrics() {
      return this.dataset.getCurrentLabelingRuleMetrics() || {};
    },
    query() {
      return this.dataset.query.text;
    },
    placeholderFields() {
      return [
        "Coverage",
        "Annotated coverage",
        "Precision",
        "Correct/incorrect",
      ];
    },
    metrics() {
      return [
        {
          name: "Coverage",
          rule: {
            value: this.formatNumber(this.ruleMetrics.coverage),
            tooltip: "Percentage of records labeled by the rule",
          },
          records: {
            value: this.$options.filters.formatNumber(
              Math.round(
                this.ruleMetrics.coverage * this.dataset.globalResults.total
              ) || 0
            ),
            total: this.$options.filters.formatNumber(
              this.dataset.globalResults.total || 0
            ),
          },
        },
        {
          name: "Annotated coverage",
          rule: {
            value: this.formatNumber(this.ruleMetrics.coverage_annotated),
            tooltip: "Percentage of annotated records labeled by the rule",
          },
          records: {
            value: this.$options.filters.formatNumber(
              Math.round(
                this.ruleMetrics.coverage_annotated *
                  this.ruleMetrics.annotated_records
              ) || 0
            ),
            total: this.$options.filters.formatNumber(
              this.ruleMetrics.annotated_records || 0
            ),
          },
        },
        {
          name: "Precision",
          rule: {
            value: this.formatNumber(this.ruleMetrics.precision),
            tooltip:
              "Percentage of correct labels given by the rule with respect to the annotations",
          },
        },
        {
          name: "Correct/incorrect",
          rule: {
            value:
              this.ruleMetrics.correct !== undefined
                ? `${this.ruleMetrics.correct}/${this.ruleMetrics.incorrect}`
                : "-/-",
            tooltip:
              "Number of labels the rule predicted correctly/incorrectly with respect to the annotations",
          },
        },
      ];
    },
  },
  methods: {
    metricsTitle(metric) {
      return metric.rule.tooltip;
    },
    formatNumber(value) {
      return isNaN(value) ? "-" : this.$options.filters.percent(value);
    },
    getValuesByMetricType(type) {
      return Object.keys(this.metricsByRules).map((key) => {
        return this.metricsByRules[key][type] || 0;
      });
    },
  },
};
</script>
<style lang="scss" scoped>
$color: #333346;
.rule-metrics {
  &__container {
    position: relative;
    display: flex;
    flex-flow: column;
    max-height: 410px;
    background: $primary-color;
    margin-left: 1em;
    color: $lighter-color;
    border-radius: $border-radius;
    margin-bottom: 2em;
    padding: 30px;
  }
  &__title {
    padding-bottom: 0;
    color: $lighter-color;
    @include font-size(22px);
    font-weight: bold;
    margin-top: 0;
  }
  &__item {
    min-height: 82px;
    .metric {
      &__rule {
        @include font-size(20px);
        font-weight: 600;
        margin-top: 0.2em;
        margin-bottom: 0.2em;
      }
      &__records {
        span {
          font-weight: 400;
        }
      }
    }
  }
}
.metric {
  &__title {
    display: inline-block;
    @include font-size(14px);
    font-weight: 600;
    margin-bottom: 0;
  }
}
p[data-title] {
  position: relative;
  @extend %hastooltip;
  &:after {
    padding: 0.5em 1em;
    bottom: 100%;
    right: 50%;
    transform: translateX(50%);
    background: $color;
    color: white;
    border: none;
    border-radius: 3px;
    @include font-size(14px);
    font-weight: 600;
    margin-bottom: 0.5em;
    min-width: 220px;
    white-space: break-spaces;
  }
  &:before {
    right: calc(50% - 7px);
    top: -0.5em;
    border-top: 7px solid $color;
    border-right: 7px solid transparent;
    border-left: 7px solid transparent;
  }
}
</style>

<style lang="scss">
.records-number {
  @include font-size(16px);
  font-weight: normal;
  .all & {
    display: none;
  }
}
</style>
