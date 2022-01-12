<script>
  import { createEventDispatcher } from "svelte";

  export let startDateMin;
  export let endDateMax;
  export let name = '';
  export let heading;
  export let labels;
  export let endDateId = 'endDate';
  export let startDateId = 'startDate';
  export let disabled = false
  export let startISODate = ''
  export let endISODate = ''

  
  let defaultLabels = {
    notSet: 'not set',
    greaterThan: 'greater than',
    lessThan: 'less than',
    range: 'range',
    day: 'day',
    days: 'days',
    apply: 'Set'
  }

  labels = {...defaultLabels, ...labels};

  if (!endDateMax) endDateMax = new Date();

  if (!startDateMin) {
    startDateMin = new Date(
      new Date().setFullYear(
        endDateMax.getFullYear(),
        endDateMax.getMonth() - 12
      )
    );
  }

  const dispatch = createEventDispatcher();

  const startTS = startISODate ? dateToTimeStamp(startISODate) : 0
  const endTS = endISODate ? dateToTimeStamp(endISODate) : 0
  const today = new Date();

  const todayRfc = timeStampToRfc(today);
  const initTimestamp = dateToTimeStamp(today);
  const startDateMinTimestamp = dateToTimeStamp(startDateMin);
  const endDateMaxTimestamp = dateToTimeStamp(endDateMax);
  const startDateMinRfc = timeStampToRfc(startDateMin);
  const endDateMaxRfc = timeStampToRfc(endDateMax);

  let sliderStartTimestamp = startTS || initTimestamp;
  let sliderEndTimestamp = endTS || initTimestamp;
  let startDate = startTS ? timeStampToRfc(startTS) : todayRfc;
  let endDate = endTS ? timeStampToRfc(endTS) : todayRfc;

  let lessThan = false;
  let greaterThan = false;
  let daysInDateRange;

  $: daysInDateRange = numberOfDaysBetweenSelectedDateRange(startDate, endDate);
  
  function dateOrSliderChange(item) {
    if (item == "endDate" && endDate && endDate < startDate)
      startDate = endDate;

    if (item == "startDate" && startDate && startDate > endDate && endDate)
      endDate = startDate;

    if (item == "endDate" || item == "startDate") {
      sliderEndTimestamp = dateToTimeStamp(endDate);
      sliderStartTimestamp = dateToTimeStamp(startDate);
    }

    if (
      (item == "sliderEndTimestamp" &&
        sliderEndTimestamp < sliderStartTimestamp) ||
      !sliderStartTimestamp
    )
      sliderStartTimestamp = sliderEndTimestamp;

    if (
      (item == "sliderStartTimestamp" &&
        sliderStartTimestamp > sliderEndTimestamp) ||
      !sliderEndTimestamp
    )
      sliderEndTimestamp = sliderStartTimestamp;

    if (item == "sliderEndTimestamp" || item == "sliderStartTimestamp") {
      endDate = timeStampToRfc(sliderEndTimestamp);
      startDate = timeStampToRfc(sliderStartTimestamp);
    }

    if (!endDate && startDate) {
      greaterThan = true;
      lessThan = false;
    }
    if (!startDate && endDate) {
      greaterThan = false;
      lessThan = true;
    }
    if (startDate && endDate) {
      lessThan = false;
      greaterThan = false;
    }
  }

  function timeStampToRfc(date) {
    if (date) return new Date(date).toJSON().slice(0, 10);
    return undefined;
  }

  function dateToTimeStamp(date) {
    if (date) return new Date(date).valueOf();
    return undefined;
  }

  function numberOfDaysBetweenSelectedDateRange(startDate, endDate) {
    if (endDate == startDate) {
      return `1 ${labels.day}`;
    } else {
      const differenceInTime =
        new Date(endDate).getTime() - new Date(startDate).getTime();
      return (differenceInTime / (1000 * 3600 * 24)).toString() + ` ${labels.days}`;
    }
  }
  
  const apply = () => {
    dispatch("onApplyDateRange", {
      startDate: startDate,
      endDate: endDate,
      name: name
    });
  };

</script>

<style>
  .applyButton {
    width: var(--applyButtonWidth, 2.2rem);
    height: var(--applyButtonHeight, 25px);
    background-color: var(--applyButtonBackgroundColor, #007bff);
    color: var(--applyButtonColor, #fff);
    padding: var(--applyButtonPadding, 5px);
    padding-right: 7px;
  }
  .applyButton.disabled {
    background-color: #CBCBCB;
  }
  .sliderEnd {
    background: var(--sliderEndBackgroundColor, #007bff);
    height: var(--sliderEndHeight, 2px);
    width: var(--sliderEndWidth, 129px);
    cursor: pointer;
    margin: var(--sliderEndMargin, 3px);
  }
  .sliderStart {
    background: var(--sliderStartBackgroundColor, #007bff);
    height: var(--sliderStartHeight, 2px);
    width: var(--sliderStartWidth, 129px);
    cursor: pointer;
    margin: var(--sliderStartMargin, 3px);
  }
  .heading {
    color: var(--headingColor, #444);
    font-size: var(--headerFontSize, 0.825em);
    font-weight: var(--headerFontWeight, 500);
  }
  .dateSelect{
    color: var(--dateSelectColor, #000);
    font-size: var(--dateSelectFontSize, 0.825em);
    font-weight: var(--dateSelectFontWeight, 300);
    height: var(--dateSelectHeight, 34px);
    width: var(--dateSelectWidth, 122px);
    margin-bottom: var(--dateSelectMarginBottom, 5px);
  }
</style>

<span class="heading">
  {heading}
  {#if !startDate && !endDate}
    {labels.notSet}
  {:else if lessThan}
    {labels.lessThan}
  {:else if greaterThan}
    {labels.greaterThan}
  {:else}
    {labels.range} {daysInDateRange}
  {/if}
</span>
<br/>

<input
  type="date"
  id={startDateId} 
  class='dateSelect'
  min={startDateMinRfc}
  max={endDateMaxRfc}
  {disabled}
  bind:value={startDate}
  on:input={() => dateOrSliderChange('startDate')}
/>
<input
  type="date"
  id={endDateId}
  class='dateSelect'
  min={startDateMinRfc}
  max={endDateMaxRfc}
  {disabled}
  bind:value={endDate}
  on:input={() => dateOrSliderChange('endDate')}
/>

<button class="applyButton" class:disabled={disabled} {disabled}
  title={labels.apply} on:click={apply}>
  {labels.apply}
</button>
<br/>

<input
  type="range"
  class="sliderStart"
  bind:value={sliderStartTimestamp}
  min={startDateMinTimestamp}
  max={endDateMaxTimestamp}
  step="86400000"
  title={new Date(startDate)}
  {disabled}
  on:input={() => dateOrSliderChange('sliderStartTimestamp')}
/>
<input
  type="range"
  class="sliderEnd"
  bind:value={sliderEndTimestamp}
  min={startDateMinTimestamp}
  max={endDateMaxTimestamp}
  step="86400000"
  title={new Date(endDate)}
  {disabled}
  on:input={() => dateOrSliderChange('sliderEndTimestamp')}
/>