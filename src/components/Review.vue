<template>

    <div>

        <h4>Confirm again your input</h4>

        <div>

<BFormGroup label-cols-sm="4" label="First Name">
    <BFormInput :name="`${prefix}_first_name`" :value="applicant.first_name" @input="v=>input(v,'applicant.first_name')" required :state="states['applicant.first_name']"></BFormInput>
            </BFormGroup>

            <BFormGroup label-cols-sm="4" label="Last Name">
                <BFormInput :name="`${prefix}_last_name`" :value="applicant.last_name" @input="v=>input(v,'applicant.last_name')" required :state="states['applicant.last_name']"></BFormInput>
            </BFormGroup>

            <BFormGroup label-cols-sm="4" label="Email">
                <BFormInput type="email" :name="`${prefix}_email`" :value="applicant.email" @input="v=>input(v,'applicant.email')" required :state="states['applicant.email']"></BFormInput>
            </BFormGroup>

            <BFormGroup label-cols-sm="4" label="Confirm your Email">
                <BFormInput type="email"
                        :name="`${prefix}_email_confirmation`"
                        :value="applicant.email_confirmation"
                        @input="v=>input(v,'applicant.email_confirmation')"
                        required
                        :state="states['applicant.email_confirmation']"
                        ></BFormInput>
            </BFormGroup>

            <BFormGroup label-cols-sm="4" label="Phone Number">
                <BFormInput type="tel" :name="`${prefix}_phone`" :value="applicant.phone" @input="v=>input(v,'applicant.phone')" required :state="states['applicant.phone']"></BFormInput>
            </BFormGroup>

            <!-- <BFormGroup label-cols-sm="4" label="Mobile Number">
                <BFormInput type="tel" :name="`${prefix}_mobile`" :value="applicant.mobile" @input="v=>input(v,'applicant.mobile')" :state="states['applicant.mobile']"></BFormInput>
            </BFormGroup> -->

            <hr />

            <BFormGroup label-cols-sm="4" label="Who provides your water service?">
                <BFormSelect
                    :name="`${prefix}_partner_id`"
                    :value="partner_id"
                    @input="v=>$emit('update:partner_id', parseInt(v) )"
                    :options="partners"
                    required
                    text-field="name"
                    value-field="id"
                    :state="states.partner_id">
                    <template slot="first">
                        <option :value="null" disabled>-- Select a city/municipality --</option>
                    </template>
                </BFormSelect>
                <spinner v-bind:busy="loading" scale=1></spinner>
            </BFormGroup>

            <BFormGroup label-cols-sm="4" label="What type of property is this?">
                <BFormSelect
                        :name="`${prefix}_property_type`"
                        :value="property.property_type"
                        @input="v=>input(v,'property.property_type')"
                        :options="$options.propertyTypes"
                        required
                        :state="states['property.property_type']"
                        >
                    <template slot="first">
                        <option :value="null" disabled>-- Select a property type --</option>
                    </template>
                </BFormSelect>
            </BFormGroup>

            <BFormGroup v-if="showBuildingTypes" label-cols-sm="4" label="Building Type">
                <BFormSelect
                        :name="`${prefix}_building_type`"
                        :value="property.building_type"
                        @input="v=>input(v,'property.building_type')"
                        :options="$options.buildingTypes"
                        required
                        :state="states['property.building_type']">
                    <template slot="first">
                        <option :value="null" disabled>-- Select a building type --</option>
                    </template>
                </BFormSelect>
            </BFormGroup>

            <BFormGroup label-cols-sm="4">
                <BFormRadioGroup
                        :name="`${prefix}_email_opt_in`"
                        :checked="applicant.email_opt_in"
                        :value="applicant.email_opt_in"
                        @input="v=>input(v,'applicant.email_opt_in')"
                        :options="$options.optInOptions"
                        :state="states['applicant.email_opt_in']"></BFormRadioGroup>
            </BFormGroup>



        </div>
        <StepTwo
            :property.sync="property"
            :application.sync="application"
            :states="states"
            @change="resetState"/>
        <StepThree
            :property.sync="property"
            :application.sync="application"
            :owner.sync="owner"
            :states="states"
            :partner="partner"
            @change="resetState"/>
        <StepFour
            :reference.sync="reference"
            :references-endpoint="referencesEndpoint"
            :applicant.sync="applicant"
            :states="states"
            @change="resetState"></StepFour>
        
    </div>

</template>

<script>
    import StepOne   from './StepOne.vue';
    import StepTwo   from './StepTwo.vue';
    import StepThree from './StepThree.vue';
    import StepFour  from './StepFour.vue';
 

    export default {

        name: 'Review',

        components: { StepOne, StepTwo, StepThree, StepFour},
        mixins: [ ],
        props: {

            applicant: {
                type: Object,
                default: () => new Applicant()
            },
            reference: {
                type: Object,
                default: () => new Reference()
            },
            referencesEndpoint: {
                type: String,
                requred: true
            },
            states: {
                type: Object,
                required: false,
                default: () => ({})     // @todo - State is not displaying
            },
            property: {
                type: Object,
                default: () => new Property()
            },
            partner_id: {
                type: [Number,String],
            },
            partnersEndpoint: {
                type: String,
                requred: true
            },
            owner: {
                type: Object,
                default: () => new Owner()
            },
            partner: {
                type: Object,
                required: true,
                default: () => new Partner()
            },
            application: {
                type: Object,
                default: () => new Application()
            },
        },
        data() {
            return {
                prefix: (Math.random()*1e32).toString(36),
                references: [],
                loading: false,
                localApplicant: JSON.parse(JSON.stringify(this.applicant))
            }
        },
        computed: {
            chosenRef() {
                return _find(this.references, {id: parseInt(this.reference.reference_type_id) });
            },
            showMoreInput() {
                return (this.chosenRef||{}).info_text || false;
            }
        },
        created() {
            this.fetchReferences();
        },
        methods: {
            resetState() {
                this.$emit('child-clicked');
            }
        }
    };

</script>