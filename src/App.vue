<template>
  <v-app>
    <v-content>
      <v-container grid-list-md>
        <v-layout row wrap>
          <v-flex xs12>
            <p class="display-3 font-weight-light">Калькулятор стоимости печати</p>
          </v-flex>
          <v-flex xs12>
            <v-card>
              <v-card-title
                primary-title
                class="headline font-weight-light"
              >Стоимость моделирования: {{ modelingTotal }}</v-card-title>
              <v-container grid-list-md>
                <v-layout row wrap>
                  <v-flex xs12 md6>
                    <v-text-field
                      label="Ставка моделирования"
                      outline
                      mask="####"
                      suffix="руб."
                      v-model="modelingCost"
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs12 md6>
                    <v-select
                      :items="modelingItems"
                      v-model="modelingItem"
                      label="Класс моделирования"
                      outline
                    ></v-select>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-card>
          </v-flex>
          <v-flex xs12>
            <v-card>
              <v-card-title
                primary-title
                class="headline font-weight-light"
              >Стоимость печати: {{printingTotal}}</v-card-title>
              <v-container grid-list-md>
                <v-layout row wrap>
                  <v-flex xs12 md6>
                    <v-text-field
                      label="Масса изделия"
                      outline
                      mask="########"
                      suffix="гр."
                      v-model="printingMass"
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs12 md6>
                    <v-select
                      :items="materialItems"
                      label="Материал печати"
                      outline
                      v-model="printingMaterial"
                    ></v-select>
                  </v-flex>
                </v-layout>
                <v-layout row wrap>
                  <v-flex xs12 md6>
                    <v-text-field
                      label="Сервисные расходы"
                      outline
                      mask="####"
                      suffix="руб./гр."
                      v-model="printingElectricityCost"
                      disabled
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs12 md6>
                    <v-select
                      :items="qualityItems"
                      label="Качество печати"
                      outline
                      v-model="printingQualitySelected"
                    ></v-select>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-card>
          </v-flex>
          <v-flex xs12>
            <v-card>
              <v-card-title
                primary-title
                class="headline font-weight-light"
              >Стоимость постобработки: {{ postworkTotal }}</v-card-title>
              <v-container grid-list-md>
                <v-layout row wrap>
                  <v-flex xs12 md6>
                    <v-text-field
                      label="Ставка постобработки"
                      outline
                      mask="####"
                      suffix="руб."
                      v-model="postworkCost"
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs12 md6>
                    <v-select
                      :items="postworkItems"
                      label="Тип постобработки"
                      outline
                      v-model="postworkTypeSelected"
                    ></v-select>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-card>
          </v-flex>
          <v-flex xs12>
            <v-card>
              <v-container grid-list-md>
                <v-layout row wrap>
                  <v-flex xs12 md6>
                    <v-text-field
                      label="Количество моделей"
                      outline
                      mask="####"
                      suffix="шт."
                      v-model="modelsCount"
                    ></v-text-field>
                  </v-flex>
                  <v-flex xs12 md6>
                    <v-text-field label="Скидка" outline mask="NNNNN" suffix="%" v-model="sale"></v-text-field>
                  </v-flex>
                  <v-flex xs12>
                    <p class="display-2 text-xs-center">Итог: {{ total }} руб.</p>
                    <p class="display-1 text-xs-center">Стоимость 1гр.: {{pricePerGr}} руб.</p>
                    <p class="text-xs-center">Выплаты сотруднику: {{ workerTotal }} руб.</p>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
  </v-app>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';

@Component
export default class App extends Vue {
  get modelingTotal(): number {
    return this.modelingCost * this.modelingItem;
  }

  get worksCost(): number {
    if (this.printingMaterial === this.qualityItems[0].value ||
      this.printingMaterial === this.qualityItems[1].value) { return 2; }
    else if (this.printingMaterial === this.qualityItems[2].value) { return 3; }
    else if (this.printingMaterial === this.qualityItems[3].value) { return 5; }
    else { return 0; }
  }

  get printingTotal(): number {
    return this.printingMass *
      ((this.printingMaterial + this.printingElectricityCost) *
        this.printingQualitySelected *
        this.printingK +
        this.worksCost);
  }
  get postworkTotal(): number {
    return this.postworkCost * this.postworkTypeSelected;
  }
  get workerTotal(): number {
    return this.modelingTotal +
      this.worksCost * this.printingMass * this.modelsCount +
      this.postworkCost * (this.postworkTypeSelected != null ? this.postworkTypeSelected : 0) * this.modelsCount;
  }
  get total(): number {
    const totalPrint = this.printingTotal * this.modelsCount < 300 ? 300 : this.printingTotal * this.modelsCount;
    return (this.modelingTotal + (this.postworkTotal * this.modelsCount) + totalPrint) * (1 - this.sale / 100);
  }
  get pricePerGr(): number {
    return this.total / this.printingMass;
  }
  public printingMaterial: number = 5;
  public printingElectricityCost: number = 1;
  public printingQualitySelected: number = 1.0;
  public qualityItems: any[] = [
    {
      text: 'Высокое (0.05 мм) (2)',
      value: 2,
    },
    {
      text: 'Хорошее (0.1мм) (1.5)',
      value: 1.5,
    },
    {
      text: 'Среднее (0.15мм) (1.2)',
      value: 1.2,
    },
    {
      text: 'Обычное (0.2 мм) (1.0)',
      value: 1.0,
    },
    {
      text: 'Грубое (>=0.3 мм) (0.8)',
      value: 0.8,
    },
  ];
  private modelingCost: number = 600;
  private modelingItems: any[] = [
    {
      text: '0 - без моделирования',
      value: 0,
    },
    {
      text: '1 - шестерни/простые',
      value: 1,
    },
    {
      text: '2 - заглушки авто/прямые формы',
      value: 2,
    },
    {
      text: '3 - заглушки дисков/плавные линии',
      value: 3,
    },
    {
      text: '4 - сложное твердотельное',
      value: 6,
    },
    {
      text: '5 - сложное поверхностное',
      value: 12,
    },
  ];
  private modelingItem: number = 0;

  private printingMass: number = 0;
  private materialItems: any[] = [
    {
      text: 'ABS/PLA (5 руб./г)',
      value: 5,
    },
    {
      text: 'ASA/SBS/PETG (7 руб./г)',
      value: 7,
    },
    {
      text: 'PA/FLEX/TPU (10 руб./г)',
      value: 10,
    },
    {
      text: 'Другие (15 руб./г)',
      value: 15,
    },
  ];
  private printingK: number = 2;

  private postworkCost: number = 500;
  private postworkTypeSelected: number = 0;
  private postworkItems: any[] = [
    { text: 'Без обработки (0)', value: 0 },
    { text: 'Удаление поддержек (0.1)', value: 0.1 },
    { text: 'Много поддержек (0.3)', value: 0.3 },
    { text: 'Химическая (1)', value: 1 },
    { text: 'Покраска (2)', value: 2 },
  ];

  private modelsCount: number = 1;
  private sale: number = 0;
}
</script>

