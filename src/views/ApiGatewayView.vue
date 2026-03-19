<template>
  <div>
    <v-row>
      <v-col cols="12" class="bg-secondary opacity-50">
        <div class="d-flex justify-space-between align-center">
          <div class="d-flex align-center">
            <v-icon class="mr-2">mdi-api</v-icon>
            <h1 class="text-h5">API Gateway</h1>
          </div>
          <v-btn color="primary" @click="openCreateDialog">
            <v-icon class="mr-2">mdi-plus</v-icon>
            Criar API
          </v-btn>
        </div>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12">
        <v-text-field
          v-model="searchQuery"
          label="Buscar APIs"
          prepend-inner-icon="mdi-magnify"
          variant="outlined"
          clearable
        >
          <template v-slot:append>
            <v-btn @click="loadAll" :loading="loading" variant="outlined">
              <v-icon class="mr-2">mdi-refresh</v-icon>
              Atualizar
            </v-btn>
          </template>
        </v-text-field>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12">
        <v-tabs v-model="tab" density="compact">
          <v-tab value="v1">REST (v1) — compatível Community</v-tab>
          <v-tab v-if="enableApigwV2" value="v2">HTTP/WebSocket (v2) — pode exigir Pro</v-tab>
        </v-tabs>
      </v-col>

      <v-col cols="12">
        <v-window v-model="tab">
          <!-- v1 -->
          <v-window-item value="v1">
            <v-row>
              <v-col cols="12" md="6" lg="4" v-for="api in filteredV1" :key="api.id">
                <v-card class="mb-4" elevation="2">
                  <v-card-title class="d-flex justify-space-between align-center">
                    <TitleNameWithTooltip :name="api.name" />
                    <v-menu>
                      <template v-slot:activator="{ props }">
                        <v-btn icon="mdi-dots-vertical" v-bind="props" size="small"></v-btn>
                      </template>
                      <v-list>
                        <v-list-item @click="openManageV1(api)">
                          <v-list-item-title>
                            <v-icon class="mr-2">mdi-cog</v-icon>
                            Gerenciar (resources / deploy)
                          </v-list-item-title>
                        </v-list-item>
                        <v-list-item @click="openDetailsV1(api)">
                          <v-list-item-title>
                            <v-icon class="mr-2">mdi-eye</v-icon>
                            Ver Detalhes
                          </v-list-item-title>
                        </v-list-item>
                        <v-list-item @click="deleteApiV1(api)">
                          <v-list-item-title>
                            <v-icon class="mr-2">mdi-delete</v-icon>
                            Deletar
                          </v-list-item-title>
                        </v-list-item>
                      </v-list>
                    </v-menu>
                  </v-card-title>

                  <v-card-text>
                    <div class="text-caption text-grey">id: {{ api.id }}</div>
                    <div class="text-caption text-grey">created: {{ api.createdDate || '—' }}</div>
                  </v-card-text>
                </v-card>
              </v-col>
            </v-row>

            <v-row v-if="!loading && apisV1.length === 0">
              <v-col cols="12" class="text-center">
                <v-icon size="64" color="grey-lighten-1">mdi-api-off</v-icon>
                <h3 class="text-h5 mt-4 text-grey">Nenhuma REST API (v1) encontrada</h3>
              </v-col>
            </v-row>
          </v-window-item>

          <!-- v2 -->
          <v-window-item v-if="enableApigwV2" value="v2">
            <v-alert type="warning" variant="tonal" class="mb-4">
              LocalStack Community pode não suportar API Gateway v2 (HTTP/WebSocket). Se aparecer erro 501, é limitação de licença/coverage.
            </v-alert>

            <v-alert v-if="v2NotSupported" type="error" variant="tonal" class="mb-4">
              API Gateway v2 retornou 501 Not Implemented no seu LocalStack. Use a aba REST (v1) ou LocalStack Pro.
            </v-alert>

            <v-row>
              <v-col cols="12" md="6" lg="4" v-for="api in filteredV2" :key="api.ApiId">
                <v-card class="mb-4" elevation="2">
                  <v-card-title class="d-flex justify-space-between align-center">
                    <TitleNameWithTooltip :name="api.Name" />
                    <v-menu>
                      <template v-slot:activator="{ props }">
                        <v-btn icon="mdi-dots-vertical" v-bind="props" size="small"></v-btn>
                      </template>
                      <v-list>
                        <v-list-item @click="openManageV2(api)">
                          <v-list-item-title>
                            <v-icon class="mr-2">mdi-cog</v-icon>
                            Gerenciar (routes / integrations / stages)
                          </v-list-item-title>
                        </v-list-item>
                        <v-list-item @click="openDetailsV2(api)">
                          <v-list-item-title>
                            <v-icon class="mr-2">mdi-eye</v-icon>
                            Ver Detalhes
                          </v-list-item-title>
                        </v-list-item>
                        <v-list-item @click="deleteApiV2(api)">
                          <v-list-item-title>
                            <v-icon class="mr-2">mdi-delete</v-icon>
                            Deletar
                          </v-list-item-title>
                        </v-list-item>
                      </v-list>
                    </v-menu>
                  </v-card-title>

                  <v-card-text>
                    <v-chip size="small" class="mr-2 mb-2" color="primary">{{ api.ProtocolType }}</v-chip>
                    <div class="text-caption text-grey">ApiId: {{ api.ApiId }}</div>
                  </v-card-text>
                </v-card>
              </v-col>
            </v-row>

            <v-row v-if="!loading && apisV2.length === 0">
              <v-col cols="12" class="text-center">
                <v-icon size="64" color="grey-lighten-1">mdi-api-off</v-icon>
                <h3 class="text-h5 mt-4 text-grey">Nenhuma API (v2) encontrada</h3>
              </v-col>
            </v-row>
          </v-window-item>
        </v-window>
      </v-col>
    </v-row>

    <!-- Create Dialog -->
    <v-dialog v-model="createDialog" max-width="650" scrollable>
      <v-card>
        <v-card-title>Criar API</v-card-title>
        <v-card-text>
          <v-select
            v-model="createForm.kind"
            label="Tipo"
            :items="createKinds"
            variant="outlined"
          />

          <v-text-field v-model="createForm.name" label="Nome *" variant="outlined" class="mt-3" />

          <template v-if="createForm.kind === 'v1'">
            <v-row class="mt-2">
              <v-col cols="12" md="4">
                <v-text-field v-model="createV1.stage" label="Stage" variant="outlined" />
              </v-col>
              <v-col cols="12" md="4">
                <v-select
                  v-model="createV1.template"
                  label="Template"
                  :items="[
                    { title: 'HTTP Proxy (mock-api)', value: 'http-proxy' },
                    { title: 'Lambda Proxy', value: 'lambda-proxy' }
                  ]"
                  variant="outlined"
                />
              </v-col>
              <v-col cols="12" md="4">
                <v-select
                  v-model="createV1.method"
                  label="Método"
                  :items="['GET','POST','PUT','PATCH','DELETE']"
                  variant="outlined"
                />
              </v-col>
            </v-row>

            <v-text-field v-model="createV1.path" label="Path" variant="outlined" class="mt-2" placeholder="/mock" />

            <template v-if="createV1.template === 'http-proxy'">
              <v-text-field v-model="createV1.httpUri" label="HTTP URI" variant="outlined" class="mt-2" />
            </template>

            <template v-else>
              <v-select
                v-model="createV1.lambdaArn"
                label="Lambda (FunctionArn)"
                :items="lambdaFunctions.map(f => ({ title: f.FunctionName, value: f.FunctionArn }))"
                variant="outlined"
                class="mt-2"
                clearable
              />
              <div class="text-caption text-grey">
                Dica: escolha <strong>apigw-test-function</strong> para um teste simples.
              </div>
            </template>
          </template>

          <template v-if="createForm.kind === 'v2'">
            <v-select
              v-model="createForm.protocolType"
              label="Protocol"
              :items="[
                { title: 'HTTP', value: 'HTTP' },
                { title: 'WEBSOCKET', value: 'WEBSOCKET' }
              ]"
              variant="outlined"
              class="mt-3"
            />
          </template>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn text @click="createDialog = false">Cancelar</v-btn>
          <v-btn
            color="primary"
            :loading="creating"
            :disabled="!createForm.name || (createForm.kind === 'v1' && createV1.template === 'lambda-proxy' && !createV1.lambdaArn)"
            @click="createApi"
          >
            Criar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Manage V1 Dialog -->
    <v-dialog v-model="manageV1Dialog" :key="selectedV1Api?.id" max-width="1100" scrollable>
      <v-card>
        <v-card-title class="d-flex justify-space-between align-center">
          <span>Gerenciar REST API (v1): {{ selectedV1Api?.name }} ({{ selectedV1Api?.id }})</span>
          <v-btn icon @click="manageV1Dialog = false"><v-icon>mdi-close</v-icon></v-btn>
        </v-card-title>

        <v-card-text style="height: 650px;">
          <!-- Row 1: Method - Path - Stage - Actions -->
          <v-row align="center">
            <v-col cols="12" md="2">
              <v-select
                v-model="testV1Method"
                label="Método"
                :items="['GET','POST','PUT','PATCH','DELETE','OPTIONS','HEAD']"
                variant="outlined"
                density="compact"
                hide-details
                :disabled="true"
              />
            </v-col>
            <v-col cols="12" md="4">
              <v-text-field
                v-model="testV1Path"
                label="Path"
                variant="outlined"
                density="compact"
                hide-details
                placeholder="/mock"
                :disabled="true"
              />
            </v-col>
            <v-col cols="12" md="2">
              <v-text-field
                v-model="v1Stage"
                label="Stage"
                variant="outlined"
                density="compact"
                hide-details
                :disabled="true"
              />
            </v-col>
            <v-col cols="12" md="4" class="d-flex justify-end ga-2">

              <v-btn color="primary" @click="executeTestV1" :loading="testingV1" :disabled="!testV1Url">
                <v-icon class="mr-2">mdi-play</v-icon>
                Testar
              </v-btn>
            </v-col>
          </v-row>

          <!-- Row 2: URL + copy -->
          <v-row>
            <v-col cols="12">
              <v-text-field
                :model-value="testV1Url"
                label="URL"
                variant="outlined"
                density="compact"
                readonly
                append-inner-icon="mdi-content-copy"
                @click:append-inner="copyToClipboard(testV1Curl)"
              />
            </v-col>
          </v-row>

          <!-- Tabs: response/body/headers/logs + time -->
          <v-row class="mt-2">
            <v-col cols="12" class="d-flex justify-space-between align-center">
              <v-tabs v-model="testV1Tab" density="compact">
                <v-tab value="response">Response</v-tab>
                <v-tab value="body">Body</v-tab>
                <v-tab value="headers">Headers</v-tab>
                <v-tab value="logs">Logs</v-tab>
              </v-tabs>
              <div class="text-caption text-grey" v-if="lastRequestMs !== null">
                {{ lastRequestMs }} ms
              </div>
            </v-col>
            <v-col cols="12">
              <v-window v-model="testV1Tab">
                <v-window-item value="response">
                  <template v-if="testV1Result">
                    <v-alert :type="testV1Result.ok ? 'success' : 'error'" variant="tonal" class="mb-3">
                      HTTP {{ testV1Result.status }} {{ testV1Result.statusText }}
                    </v-alert>
                    <v-card variant="outlined" class="pa-3">
                      <pre class="text-body-2">{{ prettyResponseV1 }}</pre>
                    </v-card>
                  </template>
                  <template v-else>
                    <div class="text-caption text-grey">Sem resposta ainda.</div>
                  </template>
                </v-window-item>

                <v-window-item value="body">
                  <v-textarea
                    v-model="testV1Body"
                    label="Body (JSON/text)"
                    rows="6"
                    variant="outlined"
                  />
                </v-window-item>

                <v-window-item value="headers">
                  <v-card variant="outlined" class="pa-3">
                    <pre class="text-body-2">{{ JSON.stringify(testV1Headers || {}, null, 2) }}</pre>
                  </v-card>
                </v-window-item>

                <v-window-item value="logs">
                  <v-card variant="outlined" class="pa-3">
                    <pre class="text-body-2">{{ testV1Logs || '—' }}</pre>
                  </v-card>
                </v-window-item>
              </v-window>
            </v-col>
          </v-row>

          <!-- Resources management hidden for now (UX simplification) -->
        </v-card-text>

      </v-card>
    </v-dialog>

    <!-- Create Resource (v1) -->
    <v-dialog v-model="createResourceDialog" max-width="700" scrollable>
      <v-card>
        <v-card-title>Criar Resource (v1)</v-card-title>
        <v-card-text>
          <v-select
            v-model="newResource.parentId"
            label="Parent"
            :items="resourcesV1.map(r => ({ title: r.path, value: r.id }))"
            variant="outlined"
          />
          <div v-if="resourcesV1.length === 0" class="text-caption text-grey mt-2">
            Nenhum resource carregado ainda. Feche e reabra o modal de Gerenciar para recarregar.
          </div>
          <v-text-field v-model="newResource.pathPart" label="PathPart * (ex: mock)" variant="outlined" class="mt-3" />
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn text @click="createResourceDialog = false">Cancelar</v-btn>
          <v-btn color="primary" :loading="manageCreating" :disabled="!newResource.pathPart || !newResource.parentId" @click="createResourceV1">
            Criar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Method/Integration (v1) -->
    <v-dialog v-model="methodDialog" max-width="900" scrollable>
      <v-card>
        <v-card-title class="d-flex justify-space-between align-center">
          <span>Configurar Method/Integration (v1): {{ selectedResource?.path }}</span>
          <v-btn icon @click="methodDialog = false"><v-icon>mdi-close</v-icon></v-btn>
        </v-card-title>
        <v-card-text>
          <v-select
            v-model="methodForm.httpMethod"
            label="HTTP Method"
            :items="['GET','POST','PUT','DELETE','PATCH','OPTIONS','HEAD']"
            variant="outlined"
          />

          <v-select
            v-model="methodForm.integrationKind"
            label="Integration"
            :items="[
              { title: 'HTTP_PROXY → mock-api', value: 'http' },
              { title: 'AWS_PROXY → Lambda', value: 'lambda' },
              { title: 'MOCK', value: 'mock' }
            ]"
            variant="outlined"
            class="mt-3"
          />

          <template v-if="methodForm.integrationKind === 'http'">
            <v-text-field v-model="methodForm.httpUri" label="HTTP URI" variant="outlined" class="mt-3" />
          </template>

          <template v-else-if="methodForm.integrationKind === 'lambda'">
            <v-select
              v-model="methodForm.lambdaArn"
              label="Lambda (FunctionArn)"
              :items="lambdaFunctions.map(f => ({ title: f.FunctionName, value: f.FunctionArn }))"
              variant="outlined"
              class="mt-3"
              clearable
            />
          </template>

          <v-alert class="mt-4" type="info" variant="tonal">
            Dica: depois de criar/alterar methods, faça deploy (aba Deploy / Test) para refletir no stage.
          </v-alert>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn text @click="methodDialog = false">Cancelar</v-btn>
          <v-btn color="primary" :loading="manageCreating" @click="applyMethodV1">
            Aplicar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Manage V2 Dialog (mantido) -->
    <v-dialog v-model="manageV2Dialog" max-width="1100" scrollable>
      <v-card>
        <v-card-title class="d-flex justify-space-between align-center">
          <span>Gerenciar API v2: {{ selectedV2Api?.Name }} ({{ selectedV2Api?.ApiId }})</span>
          <v-btn icon @click="manageV2Dialog = false"><v-icon>mdi-close</v-icon></v-btn>
        </v-card-title>

        <v-card-text style="height: 650px;">
          <v-tabs v-model="manageV2Tab" density="compact">
            <v-tab value="routes">Routes</v-tab>
            <v-tab value="integrations">Integrations</v-tab>
            <v-tab value="stages">Stages</v-tab>
          </v-tabs>

          <v-window v-model="manageV2Tab" class="mt-4">
            <v-window-item value="routes">
              <div class="d-flex justify-end mb-3">
                <v-btn color="primary" variant="outlined" @click="createRouteDialogV2 = true">
                  <v-icon class="mr-2">mdi-plus</v-icon>
                  Criar Route
                </v-btn>
              </div>

              <v-row>
                <v-col cols="12" md="6" lg="4" v-for="r in routesV2" :key="r.RouteId">
                  <v-card class="mb-4" elevation="2">
                    <v-card-title class="d-flex justify-space-between align-center">
                      <TitleNameWithTooltip :name="r.RouteKey" />
                      <div>
                        <v-btn icon="mdi-play" size="small" variant="text" @click="openTestRouteDialogV2(r)"></v-btn>
                        <v-btn icon="mdi-delete" size="small" variant="text" @click="deleteRouteV2(r)"></v-btn>
                      </div>
                    </v-card-title>
                    <v-card-text>
                      <div class="text-caption text-grey">RouteId: {{ r.RouteId }}</div>
                      <div class="text-caption text-grey">Target: {{ r.Target || '—' }}</div>
                    </v-card-text>
                  </v-card>
                </v-col>
              </v-row>
            </v-window-item>

            <v-window-item value="integrations">
              <div class="d-flex justify-end mb-3">
                <v-btn color="primary" variant="outlined" @click="createIntegrationDialogV2 = true">
                  <v-icon class="mr-2">mdi-plus</v-icon>
                  Criar Integration
                </v-btn>
              </div>

              <v-row>
                <v-col cols="12" md="6" lg="4" v-for="i in integrationsV2" :key="i.IntegrationId">
                  <v-card class="mb-4" elevation="2">
                    <v-card-title class="d-flex justify-space-between align-center">
                      <span class="text-subtitle-1">{{ i.IntegrationType }}</span>
                      <v-btn icon="mdi-delete" size="small" variant="text" @click="deleteIntegrationV2(i)"></v-btn>
                    </v-card-title>
                    <v-card-text>
                      <div class="text-caption text-grey">IntegrationId: {{ i.IntegrationId }}</div>
                      <div class="text-caption text-grey">Uri: {{ i.IntegrationUri || '—' }}</div>
                    </v-card-text>
                  </v-card>
                </v-col>
              </v-row>
            </v-window-item>

            <v-window-item value="stages">
              <div class="d-flex justify-end mb-3">
                <v-btn color="primary" variant="outlined" @click="createStageDialogV2 = true">
                  <v-icon class="mr-2">mdi-plus</v-icon>
                  Criar Stage
                </v-btn>
              </div>

              <v-row>
                <v-col cols="12" md="6" lg="4" v-for="s in stagesV2" :key="s.StageName">
                  <v-card class="mb-4" elevation="2">
                    <v-card-title class="d-flex justify-space-between align-center">
                      <TitleNameWithTooltip :name="s.StageName" />
                      <v-btn icon="mdi-delete" size="small" variant="text" @click="deleteStageV2(s)"></v-btn>
                    </v-card-title>
                  </v-card>
                </v-col>
              </v-row>
            </v-window-item>
          </v-window>
        </v-card-text>

      </v-card>
    </v-dialog>

    <!-- Test Route V2 Dialog -->
    <v-dialog v-model="testRouteDialogV2" max-width="900" scrollable>
      <v-card>
        <v-card-title class="d-flex justify-space-between align-center">
          <span>Testar Route (v2): {{ routeToTestV2?.RouteKey }}</span>
          <v-btn icon @click="testRouteDialogV2 = false"><v-icon>mdi-close</v-icon></v-btn>
        </v-card-title>
        <v-card-text>
          <v-row>
            <v-col cols="12" md="6">
              <v-select v-model="testStageV2" label="Stage" :items="stagesV2.map(s => s.StageName)" variant="outlined" />
            </v-col>
          </v-row>

          <v-card variant="outlined" class="pa-3">
            <div class="text-subtitle-2 mb-2">URL</div>
            <code class="text-body-2">{{ testUrlV2 }}</code>
          </v-card>

          <div class="d-flex justify-end mt-4">
            <v-btn color="primary" :loading="testingV2" :disabled="!testUrlV2" @click="executeTestV2">
              <v-icon class="mr-2">mdi-play</v-icon>
              Testar agora
            </v-btn>
          </div>

          <template v-if="testV2Result">
            <v-divider class="my-4" />
            <v-alert :type="testV2Result.ok ? 'success' : 'error'" variant="tonal" class="mb-3">
              HTTP {{ testV2Result.status }} {{ testV2Result.statusText }}
            </v-alert>
            <v-card variant="outlined" class="pa-3">
              <div class="text-subtitle-2 mb-2">Response body</div>
              <pre class="text-body-2">{{ testV2Result.body }}</pre>
            </v-card>
          </template>
        </v-card-text>
      </v-card>
    </v-dialog>

    <!-- Details Dialog -->
    <v-dialog v-model="detailsDialog" max-width="900" scrollable>
      <v-card>
        <v-card-title class="d-flex justify-space-between align-center">
          <span>Detalhes</span>
          <v-btn icon @click="detailsDialog = false"><v-icon>mdi-close</v-icon></v-btn>
        </v-card-title>
        <v-card-text style="height: 550px;">
          <v-card variant="outlined" class="pa-3">
            <pre class="text-body-2">{{ JSON.stringify(detailsData, null, 2) }}</pre>
          </v-card>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'
import { useAppStore } from '@/stores/app'
import { storeToRefs } from 'pinia'
import TitleNameWithTooltip from '@/components/TitleNameWithTooltip.vue'

// v2
import {
  GetApisCommand,
  GetApiCommand,
  CreateApiCommand,
  DeleteApiCommand,
  GetRoutesCommand,
  CreateRouteCommand,
  DeleteRouteCommand,
  GetIntegrationsCommand,
  CreateIntegrationCommand,
  DeleteIntegrationCommand,
  GetStagesCommand,
  CreateStageCommand,
  DeleteStageCommand
} from '@aws-sdk/client-apigatewayv2'

// v1
import {
  GetRestApisCommand,
  GetRestApiCommand,
  CreateRestApiCommand,
  DeleteRestApiCommand,
  GetResourcesCommand,
  CreateResourceCommand,
  DeleteResourceCommand,
  PutMethodCommand,
  PutIntegrationCommand,
  CreateDeploymentCommand
} from '@aws-sdk/client-api-gateway'

import { ListFunctionsCommand } from '@aws-sdk/client-lambda'

const appStore = useAppStore()
const { apigateway, apigatewayv2, lambda } = storeToRefs(appStore)

const enableApigwV2 = (import.meta.env.VITE_ENABLE_APIGW_V2 || 'false') === 'true'

const tab = ref('v1')
const loading = ref(false)
const creating = ref(false)
const searchQuery = ref('')

const apisV2 = ref([])
const apisV1 = ref([])

const v2NotSupported = ref(false)

const createDialog = ref(false)

const loadLambdaOptions = async () => {
  try {
    const lambdas = await lambda.value.send(new ListFunctionsCommand({}))
    lambdaFunctions.value = lambdas.Functions || []

    // if apigw-test-function exists, preselect it
    if (!createV1.value.lambdaArn) {
      const preferred = (lambdaFunctions.value || []).find(f => f.FunctionName === 'apigw-test-function')
      if (preferred?.FunctionArn) createV1.value.lambdaArn = preferred.FunctionArn
    }
  } catch (e) {
    console.error('Error loading lambdas:', e)
  }
}

const openCreateDialog = async () => {
  createDialog.value = true
  await loadLambdaOptions()
}

const createForm = ref({ kind: 'v1', name: '', protocolType: 'HTTP' })

// v1 wizard defaults
const createV1 = ref({
  stage: 'dev',
  template: 'http-proxy', // http-proxy | lambda-proxy
  method: 'GET',
  path: '/mock',
  httpUri: 'http://mock-api',
  lambdaArn: ''
})

// Helps pick default test path for newly-created APIs (best-effort; no persistence)
const createdV1DefaultsByApiId = ref({})
const createdV1DefaultsByApiName = ref({})

const createKinds = computed(() => {
  const items = [{ title: 'REST (v1)', value: 'v1' }]
  if (enableApigwV2) items.push({ title: 'HTTP/WebSocket (v2)', value: 'v2' })
  return items
})

const detailsDialog = ref(false)
const detailsData = ref(null)

// ------- v1 manage -------
const manageV1Dialog = ref(false)
// manageV1Tab removed (unified UX)
const selectedV1Api = ref(null)
const resourcesV1 = ref([])
const rootResourceId = ref(null)

const lambdaFunctions = ref([])

const createResourceDialog = ref(false)
const newResource = ref({ parentId: '', pathPart: '' })

const methodDialog = ref(false)
const selectedResource = ref(null)
const methodForm = ref({
  httpMethod: 'GET',
  integrationKind: 'http',
  httpUri: 'http://mock-api',
  lambdaArn: ''
})

const v1Stage = ref('dev')
const testV1Method = ref('GET')
const testV1Path = ref('/mock')
const testV1Body = ref('')
const testingV1 = ref(false)

// Samples are currently read-only; v1 test form is also read-only after creation (edit only in the wizard)

const lastRequestMs = ref(null)
const testV1Result = ref(null)
const testV1Headers = ref(null)
const testV1Logs = ref('')
const testV1Tab = ref('response')

const testV1Url = computed(() => {
  if (!selectedV1Api.value?.id) return ''
  const base = (appStore.endpoint || '').replace(/\/$/, '')
  const stage = (v1Stage.value || 'dev').trim()
  const path = (testV1Path.value || '/').trim()
  const normalized = path.startsWith('/') ? path : `/${path}`
  return `${base}/restapis/${selectedV1Api.value.id}/${stage}/_user_request_${normalized}`
})

const testV1Curl = computed(() => {
  if (!testV1Url.value) return ''

  const method = (testV1Method.value || 'GET').toUpperCase()
  const hasBody = ['POST', 'PUT', 'PATCH'].includes(method) && !!testV1Body.value

  if (hasBody) {
    const bodyEscaped = testV1Body.value.replace(/'/g, "'\\''")
    return `curl -i -X ${method} -H 'Content-Type: application/json' --data '${bodyEscaped}' "${testV1Url.value}"`
  }

  if (method !== 'GET') return `curl -i -X ${method} "${testV1Url.value}"`

  return `curl -i "${testV1Url.value}"`
})

const prettyResponseV1 = computed(() => {
  const raw = testV1Result.value?.body
  if (!raw) return ''
  try {
    const parsed = JSON.parse(raw)
    return JSON.stringify(parsed, null, 2)
  } catch {
    return raw
  }
})

// ------- v2 manage (best effort) -------
const manageV2Dialog = ref(false)
const manageV2Tab = ref('routes')
const selectedV2Api = ref(null)
const routesV2 = ref([])
const integrationsV2 = ref([])
const stagesV2 = ref([])

const createRouteDialogV2 = ref(false)
const createIntegrationDialogV2 = ref(false)
const createStageDialogV2 = ref(false)

const testRouteDialogV2 = ref(false)
const routeToTestV2 = ref(null)
const testStageV2 = ref('$default')
const testingV2 = ref(false)
const testV2Result = ref(null)

const testUrlV2 = computed(() => {
  if (!selectedV2Api.value || !routeToTestV2.value) return ''
  const base = (appStore.endpoint || '').replace(/\/$/, '')
  const apiId = selectedV2Api.value.ApiId
  const stage = testStageV2.value || '$default'
  const parts = String(routeToTestV2.value.RouteKey || '').split(' ')
  const path = parts.length >= 2 ? parts.slice(1).join(' ') : '/'
  const normalizedPath = path.startsWith('/') ? path : `/${path}`
  return `${base}/_aws/execute-api/${apiId}/${stage}${normalizedPath}`
})

const filteredV2 = computed(() => {
  const q = (searchQuery.value || '').toLowerCase().trim()
  if (!q) return apisV2.value
  return apisV2.value.filter(a => (a.Name || '').toLowerCase().includes(q) || (a.ApiId || '').toLowerCase().includes(q))
})

const filteredV1 = computed(() => {
  const q = (searchQuery.value || '').toLowerCase().trim()
  if (!q) return apisV1.value
  return apisV1.value.filter(a => (a.name || '').toLowerCase().includes(q) || (a.id || '').toLowerCase().includes(q))
})

const loadV2 = async () => {
  if (!enableApigwV2) {
    apisV2.value = []
    return
  }

  try {
    const resp = await apigatewayv2.value.send(new GetApisCommand({}))
    apisV2.value = resp.Items || []
    v2NotSupported.value = false
  } catch (error) {
    // LocalStack returns 501 for Pro-only features
    const status = error?.$metadata?.httpStatusCode
    if (status === 501) v2NotSupported.value = true
    apisV2.value = []
  }
}

const loadV1 = async () => {
  const resp = await apigateway.value.send(new GetRestApisCommand({ limit: 500 }))
  apisV1.value = resp.items || []
}

const loadAll = async () => {
  loading.value = true
  try {
    await Promise.all([loadV1(), loadV2()])
  } catch (error) {
    console.error('Error loading apis:', error)
    appStore.showSnackbar('Erro ao carregar APIs', 'error')
  } finally {
    loading.value = false
  }
}

const createApi = async () => {
  if (!createForm.value.name) return

  creating.value = true
  try {
    if (createForm.value.kind === 'v1') {
      // Wizard: create usable API (resource + method + integration + deployment)
      const created = await apigateway.value.send(new CreateRestApiCommand({ name: createForm.value.name }))
      const restApiId = created.id

      const stageName = (createV1.value.stage || 'dev').trim()
      const method = (createV1.value.method || 'GET').toUpperCase()
      const path = (createV1.value.path || '/').trim()
      const normalized = path.startsWith('/') ? path : `/${path}`
      const pathPart = normalized.replace(/^\//, '')

      // store defaults for later modal open
      createdV1DefaultsByApiId.value[restApiId] = {
        stage: stageName,
        method,
        path: normalized
      }
      createdV1DefaultsByApiName.value[(createForm.value.name || '').toLowerCase().trim()] = {
        stage: stageName,
        method,
        path: normalized
      }

      if (!pathPart || pathPart.includes('/')) {
        throw new Error('Por enquanto o wizard aceita apenas path de 1 nível (ex: /mock)')
      }

      const resources = await apigateway.value.send(new GetResourcesCommand({ restApiId, limit: 500 }))
      const root = (resources.items || []).find(r => r.path === '/')
      if (!root?.id) throw new Error('Root resource não encontrado')

      const newRes = await apigateway.value.send(new CreateResourceCommand({
        restApiId,
        parentId: root.id,
        pathPart
      }))

      const resourceId = newRes.id

      await apigateway.value.send(new PutMethodCommand({
        restApiId,
        resourceId,
        httpMethod: method,
        authorizationType: 'NONE'
      }))

      if (createV1.value.template === 'http-proxy') {
        const uri = (createV1.value.httpUri || 'http://mock-api').trim()
        await apigateway.value.send(new PutIntegrationCommand({
          restApiId,
          resourceId,
          httpMethod: method,
          type: 'HTTP_PROXY',
          integrationHttpMethod: method,
          uri
        }))
      } else {
        const lambdaArn = createV1.value.lambdaArn
        const uri = `arn:aws:apigateway:${appStore.region || 'us-east-1'}:lambda:path/2015-03-31/functions/${lambdaArn}/invocations`
        await apigateway.value.send(new PutIntegrationCommand({
          restApiId,
          resourceId,
          httpMethod: method,
          type: 'AWS_PROXY',
          integrationHttpMethod: 'POST',
          uri
        }))
      }

      await apigateway.value.send(new CreateDeploymentCommand({ restApiId, stageName }))

      appStore.showSnackbar('REST API (v1) criada e publicada!', 'success')
      tab.value = 'v1'
    } else {
      await apigatewayv2.value.send(new CreateApiCommand({ Name: createForm.value.name, ProtocolType: createForm.value.protocolType }))
      appStore.showSnackbar('API (v2) criada!', 'success')
      tab.value = 'v2'
    }

    createDialog.value = false
    createForm.value.name = ''
    await loadAll()
  } catch (error) {
    console.error('Error creating api:', error)
    appStore.showSnackbar('Erro ao criar API', 'error')
  } finally {
    creating.value = false
  }
}

const openDetailsV2 = async (api) => {
  try {
    const resp = await apigatewayv2.value.send(new GetApiCommand({ ApiId: api.ApiId }))
    detailsData.value = resp
    detailsDialog.value = true
  } catch (error) {
    console.error('Error getting api details v2:', error)
    appStore.showSnackbar('Erro ao carregar detalhes (v2)', 'error')
  }
}

const openDetailsV1 = async (api) => {
  try {
    const resp = await apigateway.value.send(new GetRestApiCommand({ restApiId: api.id }))
    detailsData.value = resp
    detailsDialog.value = true
  } catch (error) {
    console.error('Error getting api details v1:', error)
    appStore.showSnackbar('Erro ao carregar detalhes (v1)', 'error')
  }
}

const deleteApiV2 = async (api) => {
  if (!confirm(`Deseja realmente deletar a API (v2) "${api.Name}"?`)) return

  try {
    await apigatewayv2.value.send(new DeleteApiCommand({ ApiId: api.ApiId }))
    appStore.showSnackbar('API (v2) deletada!', 'success')
    await loadAll()
  } catch (error) {
    console.error('Error deleting api v2:', error)
    appStore.showSnackbar('Erro ao deletar API (v2)', 'error')
  }
}

const deleteApiV1 = async (api) => {
  if (!confirm(`Deseja realmente deletar a API (v1) "${api.name}"?`)) return

  try {
    await apigateway.value.send(new DeleteRestApiCommand({ restApiId: api.id }))
    appStore.showSnackbar('API (v1) deletada!', 'success')
    await loadAll()
  } catch (error) {
    console.error('Error deleting api v1:', error)
    appStore.showSnackbar('Erro ao deletar API (v1)', 'error')
  }
}

const reloadManageV1 = async () => {
  if (!selectedV1Api.value) return

  manageLoading.value = true
  try {
    const [res, lambdas] = await Promise.all([
      apigateway.value.send(new GetResourcesCommand({ restApiId: selectedV1Api.value.id, limit: 500 })),
      lambda.value.send(new ListFunctionsCommand({}))
    ])

    resourcesV1.value = res.items || []
    const root = resourcesV1.value.find(r => r.path === '/')
    rootResourceId.value = root?.id || null

    newResource.value.parentId = rootResourceId.value || ''

    lambdaFunctions.value = lambdas.Functions || []
  } catch (error) {
    console.error('Error loading v1 resources:', error)
    appStore.showSnackbar('Erro ao carregar resources (v1)', 'error')
  } finally {
    manageLoading.value = false
  }
}

const setDefaultTestForApi = () => {
  // Defaults per API name
  v1Stage.value = 'dev'
  testV1Method.value = 'GET'
  testV1Body.value = ''

  const name = (selectedV1Api.value?.name || '').toLowerCase().trim()
  if (name.includes('sample-rest-api-lambda')) {
    testV1Path.value = '/apigw-lambda'
    return
  }
  if (name.includes('sample-rest-api-simple')) {
    testV1Path.value = '/mock'
    return
  }

  // If created via wizard in this runtime, prefer the known path
  const created =
    createdV1DefaultsByApiId.value[selectedV1Api.value?.id] ||
    createdV1DefaultsByApiName.value[(selectedV1Api.value?.name || '').toLowerCase().trim()]

  if (created?.path) {
    v1Stage.value = created.stage || v1Stage.value
    testV1Method.value = created.method || testV1Method.value
    testV1Path.value = created.path
    return
  }

  // Fallback: pick first non-root resource path
  const nonRoot = (resourcesV1.value || []).filter(r => r.path && r.path !== '/')
  if (nonRoot.length > 0) {
    testV1Path.value = nonRoot[0].path.startsWith('/') ? nonRoot[0].path : `/${nonRoot[0].path}`

    // pick method if resource has it
    const methods = Object.keys(nonRoot[0].resourceMethods || {})
    if (methods.length > 0) testV1Method.value = methods[0]

    return
  }

  testV1Path.value = '/'
}

const normalizeTestV1Path = () => {
  const p = (testV1Path.value || '').trim()
  testV1Path.value = p ? (p.startsWith('/') ? p : `/${p}`) : '/'
}

const openManageV1 = async (api) => {
  // Force a full remount of the modal content when switching APIs.
  // This avoids Vuetify reusing internal input state across different APIs.
  manageV1Dialog.value = false
  await nextTick()

  selectedV1Api.value = api

  // clear data that could affect default selection
  resourcesV1.value = []
  rootResourceId.value = null

  // reset runtime-only outputs
  testV1Result.value = null
  testV1Headers.value = null
  testV1Logs.value = ''
  lastRequestMs.value = null
  testV1Tab.value = 'response'

  // reset form fields BEFORE loading (so UI doesn't flash old values)
  v1Stage.value = 'dev'
  testV1Method.value = 'GET'
  testV1Path.value = '/'
  testV1Body.value = ''

  manageV1Dialog.value = true
  await nextTick()

  // Pre-set sample/created paths ASAP (even if GetResources fails)
  const initialName = (api?.name || '').toLowerCase().trim()
  if (initialName.includes('sample-rest-api-lambda')) testV1Path.value = '/apigw-lambda'
  if (initialName.includes('sample-rest-api-simple')) testV1Path.value = '/mock'

  const created =
    createdV1DefaultsByApiId.value[api?.id] ||
    createdV1DefaultsByApiName.value[initialName]

  if (created?.path) {
    v1Stage.value = created.stage || v1Stage.value
    testV1Method.value = created.method || testV1Method.value
    testV1Path.value = created.path
  }

  await reloadManageV1()
  setDefaultTestForApi()
  normalizeTestV1Path()

}

const createResourceV1 = async () => {
  if (!selectedV1Api.value || !newResource.value.parentId || !newResource.value.pathPart) return

  manageCreating.value = true
  try {
    await apigateway.value.send(new CreateResourceCommand({
      restApiId: selectedV1Api.value.id,
      parentId: newResource.value.parentId,
      pathPart: newResource.value.pathPart
    }))

    appStore.showSnackbar('Resource criado!', 'success')
    createResourceDialog.value = false
    newResource.value.pathPart = ''
    await reloadManageV1()
  } catch (error) {
    console.error('Error creating resource v1:', error)
    appStore.showSnackbar('Erro ao criar resource', 'error')
  } finally {
    manageCreating.value = false
  }
}

const deleteResourceV1 = async (res) => {
  if (!selectedV1Api.value) return
  if (res.path === '/') return
  if (!confirm(`Deletar resource "${res.path}"?`)) return

  try {
    await apigateway.value.send(new DeleteResourceCommand({ restApiId: selectedV1Api.value.id, resourceId: res.id }))
    appStore.showSnackbar('Resource deletado!', 'success')
    await reloadManageV1()
  } catch (error) {
    console.error('Error deleting resource v1:', error)
    appStore.showSnackbar('Erro ao deletar resource', 'error')
  }
}

const openMethodDialog = (res) => {
  selectedResource.value = res
  methodForm.value = {
    httpMethod: 'GET',
    integrationKind: 'http',
    httpUri: 'http://mock-api',
    lambdaArn: ''
  }
  methodDialog.value = true
}

const applyMethodV1 = async () => {
  if (!selectedV1Api.value || !selectedResource.value) return

  manageCreating.value = true
  try {
    const restApiId = selectedV1Api.value.id
    const resourceId = selectedResource.value.id
    const httpMethod = methodForm.value.httpMethod

    await apigateway.value.send(new PutMethodCommand({
      restApiId,
      resourceId,
      httpMethod,
      authorizationType: 'NONE'
    }))

    if (methodForm.value.integrationKind === 'mock') {
      await apigateway.value.send(new PutIntegrationCommand({
        restApiId,
        resourceId,
        httpMethod,
        type: 'MOCK'
      }))
    } else if (methodForm.value.integrationKind === 'http') {
      await apigateway.value.send(new PutIntegrationCommand({
        restApiId,
        resourceId,
        httpMethod,
        type: 'HTTP_PROXY',
        integrationHttpMethod: httpMethod,
        uri: methodForm.value.httpUri
      }))
    } else {
      // lambda
      const lambdaArn = methodForm.value.lambdaArn
      if (!lambdaArn) {
        appStore.showSnackbar('Selecione uma Lambda', 'warning')
        return
      }

      const uri = `arn:aws:apigateway:${appStore.region || 'us-east-1'}:lambda:path/2015-03-31/functions/${lambdaArn}/invocations`

      await apigateway.value.send(new PutIntegrationCommand({
        restApiId,
        resourceId,
        httpMethod,
        type: 'AWS_PROXY',
        integrationHttpMethod: 'POST',
        uri
      }))
    }

    appStore.showSnackbar('Method/Integration aplicado!', 'success')
    methodDialog.value = false
    await reloadManageV1()
  } catch (error) {
    console.error('Error applying method v1:', error)
    appStore.showSnackbar('Erro ao aplicar method/integration', 'error')
  } finally {
    manageCreating.value = false
  }
}

const deployV1 = async () => {
  if (!selectedV1Api.value) return

  manageLoading.value = true
  try {
    await apigateway.value.send(new CreateDeploymentCommand({
      restApiId: selectedV1Api.value.id,
      stageName: v1Stage.value || 'dev'
    }))
    appStore.showSnackbar('Deployment criado!', 'success')
  } catch (error) {
    console.error('Error creating deployment v1:', error)
    appStore.showSnackbar('Erro ao criar deployment', 'error')
  } finally {
    manageLoading.value = false
  }
}

const copyToClipboard = async (text) => {
  try {
    await navigator.clipboard.writeText(text || '')
    appStore.showSnackbar('Copiado!', 'success')
  } catch (error) {
    console.error('Clipboard error:', error)
    appStore.showSnackbar('Falha ao copiar', 'error')
  }
}

const executeTestV1 = async () => {
  if (!testV1Url.value) return

  testingV1.value = true
  testV1Result.value = null
  testV1Headers.value = null
  testV1Logs.value = ''
  lastRequestMs.value = null

  try {
    const method = (testV1Method.value || 'GET').toUpperCase()
    const hasBody = ['POST', 'PUT', 'PATCH'].includes(method) && !!testV1Body.value

    const startedAt = performance.now()

    const resp = await fetch(testV1Url.value, {
      method,
      headers: hasBody ? { 'Content-Type': 'application/json' } : undefined,
      body: hasBody ? testV1Body.value : undefined
    })

    const body = await resp.text().catch(() => '')
    const endedAt = performance.now()

    lastRequestMs.value = Math.round(endedAt - startedAt)

    const headersObj = {}
    try {
      for (const [k, v] of resp.headers.entries()) headersObj[k] = v
    } catch {}

    testV1Headers.value = headersObj
    testV1Logs.value = ''
    testV1Tab.value = 'response'

    testV1Result.value = { ok: resp.ok, status: resp.status, statusText: resp.statusText, body }
  } catch (error) {
    console.error('Error testing v1:', error)
    const msg = error?.message ? `Erro ao testar (v1): ${error.message}` : 'Erro ao testar (v1)'
    appStore.showSnackbar(msg, 'error', 8000)
  } finally {
    testingV1.value = false
  }
}

// --- v2 manage (best effort, may 501) ---
const reloadManageV2 = async () => {
  if (!selectedV2Api.value) return

  manageLoading.value = true
  try {
    const apiId = selectedV2Api.value.ApiId

    const [routes, integrations, stages, lambdas] = await Promise.all([
      apigatewayv2.value.send(new GetRoutesCommand({ ApiId: apiId })),
      apigatewayv2.value.send(new GetIntegrationsCommand({ ApiId: apiId })),
      apigatewayv2.value.send(new GetStagesCommand({ ApiId: apiId })),
      lambda.value.send(new ListFunctionsCommand({}))
    ])

    routesV2.value = routes.Items || []
    integrationsV2.value = integrations.Items || []
    stagesV2.value = stages.Items || []
    lambdaFunctions.value = lambdas.Functions || []
  } catch (error) {
    console.error('Error reloading manage v2:', error)
    appStore.showSnackbar('Erro ao carregar dados (v2)', 'error')
  } finally {
    manageLoading.value = false
  }
}

const openManageV2 = async (api) => {
  selectedV2Api.value = api
  manageV2Tab.value = 'routes'
  manageV2Dialog.value = true
  await reloadManageV2()
}

const deleteRouteV2 = async (route) => {
  if (!selectedV2Api.value) return
  if (!confirm(`Deseja deletar a route "${route.RouteKey}"?`)) return

  try {
    await apigatewayv2.value.send(new DeleteRouteCommand({ ApiId: selectedV2Api.value.ApiId, RouteId: route.RouteId }))
    appStore.showSnackbar('Route deletada!', 'success')
    await reloadManageV2()
  } catch (error) {
    console.error('Error deleting route v2:', error)
    appStore.showSnackbar('Erro ao deletar route (v2)', 'error')
  }
}

const deleteIntegrationV2 = async (integration) => {
  if (!selectedV2Api.value) return
  if (!confirm(`Deseja deletar a integration "${integration.IntegrationId}"?`)) return

  try {
    await apigatewayv2.value.send(new DeleteIntegrationCommand({ ApiId: selectedV2Api.value.ApiId, IntegrationId: integration.IntegrationId }))
    appStore.showSnackbar('Integration deletada!', 'success')
    await reloadManageV2()
  } catch (error) {
    console.error('Error deleting integration v2:', error)
    appStore.showSnackbar('Erro ao deletar integration (v2)', 'error')
  }
}

const deleteStageV2 = async (stage) => {
  if (!selectedV2Api.value) return
  if (stage.StageName === '$default') {
    appStore.showSnackbar('Não vou deletar o stage "$default"', 'warning')
    return
  }
  if (!confirm(`Deseja deletar o stage "${stage.StageName}"?`)) return

  try {
    await apigatewayv2.value.send(new DeleteStageCommand({ ApiId: selectedV2Api.value.ApiId, StageName: stage.StageName }))
    appStore.showSnackbar('Stage deletado!', 'success')
    await reloadManageV2()
  } catch (error) {
    console.error('Error deleting stage v2:', error)
    appStore.showSnackbar('Erro ao deletar stage (v2)', 'error')
  }
}

const openTestRouteDialogV2 = (route) => {
  routeToTestV2.value = route
  const stageNames = (stagesV2.value || []).map(s => s.StageName)
  testStageV2.value = stageNames.includes('$default') ? '$default' : stageNames[0] || 'dev'
  testV2Result.value = null
  testRouteDialogV2.value = true
}

const openCreateResourceDialog = async () => {
  // ensure resources are loaded so Parent dropdown is populated
  if (!resourcesV1.value || resourcesV1.value.length === 0) {
    await reloadManageV1()
  }

  newResource.value.parentId = rootResourceId.value || resourcesV1.value[0]?.id || ''
  createResourceDialog.value = true
}

// no per-api persistence; keep each open isolated

const executeTestV2 = async () => {
  if (!testUrlV2.value) return

  testingV2.value = true
  testV2Result.value = null

  try {
    const method = String(routeToTestV2.value?.RouteKey || 'GET').split(' ')[0] || 'GET'
    const resp = await fetch(testUrlV2.value, { method: method === '$default' ? 'GET' : method })
    const body = await resp.text()
    testV2Result.value = { ok: resp.ok, status: resp.status, statusText: resp.statusText, body }
  } catch (error) {
    console.error('Error testing v2:', error)
    appStore.showSnackbar('Erro ao testar (v2)', 'error')
  } finally {
    testingV2.value = false
  }
}

// no session persistence for v1 test form

onMounted(() => {
  loadAll()
})
</script>
