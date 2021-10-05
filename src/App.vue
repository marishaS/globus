<template>
    <div id="app">
        <div v-if="error">Error: {{ error }}</div>
        <div style="overflow-x:auto;">
            <table>
                <thead>
                    <th>status</th>
                    <th>progress</th>
                    <th>user</th>
                    <th>request date</th>
                </thead>
                <tbody>
                    <tr v-for="data in globusData" :key="data.id">
                        <td>
                            <span v-if="data.customStatus === 'INACTIVE'">not started</span>
                            <span v-if="data.customStatus === 'ERROR'">
                                Halted: {{ formatDate(data.end_date) }}
                            </span>
                            <span v-if="data.customStatus === 'SUCCESS'">
                                Completed: {{ formatDate(data.end_date) }}
                            </span>
                            <span v-if="data.customStatus === 'IN PROGRESS'">
                                Time Remaining: {{ formatRemainingTime(data.remaining) }}
                            </span>
                            <br>
                            <p v-html="boldTheStatus(data.status)"></p>
                        </td>
                        <td>{{ getProgress(data) }}</td>
                        <td><a :href="'mailto:'+ data.email">{{ data.fullname ? data.fullname : "N/A" }}</a></td>
                        <td>{{ formatDate(data.request_date) }}</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script>
const moment = require('moment');
import axios from "axios";
export default {
    name: 'GlobusHome',
    data: function () {
        return {
            globusData : [],
            error: null,
            sortOrder: {
                "INACTIVE": 1,
                "IN PROGRESS": 2,
                "SUCCESS": 3,
                "ERROR": 4
            }
        }
    },
    created(){
        this.getData();
    },
    methods: {
        async getData() {
            await axios.get(process.env.VUE_APP_API_ENDPOINT + '1', {
                headers: {
                    'api-key': 'globus'
                }
            }).then(response => {
                try {
                    if (JSON.parse(JSON.stringify(response.data)) && JSON.parse(JSON.stringify(response.data.DATA )))
                    {
                        this.globusData = JSON.parse(JSON.stringify(response.data.DATA));
                        this.globusData.forEach(item => {
                            if(!this.hasStart(item)) {
                                item.customStatus = "INACTIVE";
                            } else if (this.hasStart(item) && this.hasEndDate(item) && !this.totalEqualsProcessed(item)) {
                                item.customStatus = "ERROR";
                            } else if(this.hasStart(item) && this.hasEndDate(item) && this.totalEqualsProcessed(item)) {
                                item.customStatus = "SUCCESS";
                            } else if(this.hasStart(item) && !this.hasEndDate(item) && !this.totalEqualsProcessed(item)) {
                                item.customStatus = "IN PROGRESS";
                            }
                        });
                        this.globusData.sort((a, b) => {
                            // Sort as per the order defined in sortOrder object.
                            return this.sortOrder[a.customStatus] - this.sortOrder[b.customStatus];
                        }).sort((a,b) => {
                            // Sort by completion date if Task completed
                            if(a.customStatus === "SUCCESS" && b.customStatus === "SUCCESS") {
                                return new Date(a.end_date) - new Date(b.end_date);
                            }
                        })
                    }
                }
                catch (error) {
                    this.error = "Could not parse data!";
                }
            }).catch(error => {
                this.error = error;
            }).finally(() => {
                console.log("Done Loading!")
            });
        },
        formatDate(date) {
            return moment(date).format("MM/DD/YYYY hh:mm A")
        },
        getProgress(report) {
            return this.bytesToGB(report.processed) + '/' + this.bytesToGB(report.total);
        },
        bytesToGB(bytesValue, decimals=2) {
            if (bytesValue === 0) return '0 b';
            if(isNaN(bytesValue)) return '0 b';
            const k = 1024;
            const dm = decimals < 0 ? 0 : decimals;
            const sizes = ['b', 'KB', 'MB', 'GB'];
            const i = Math.floor(Math.log(bytesValue) / Math.log(k));
            return parseFloat((bytesValue / Math.pow(k, i)).toFixed(dm)) + ' ' + sizes[i];
        },
        hasStart(report) {
            return report.start_date !== undefined;
        },
        hasEndDate(report) {
            return report.end_date !== undefined;
        },
        totalEqualsProcessed(report) {
            return report.total === report.processed;
        },
        formatRemainingTime(remaining) {
            const seconds = Number(remaining/1000);
            const d = Number(seconds);
            const h = Math.floor(d / 3600);
            const m = Math.floor(d % 3600 / 60);
            const s = Math.floor(d % 3600 % 60);
            if(h <= 48) return h + ':' + m + ':' + s;
            return Math.floor(h/24) + ' days';
        },
        boldTheStatus(status) {
            if(status.toLowerCase().includes('success')) {
                return status.replace('success', "<b>success</b>")
                .replace('SUCCESS', "<b>SUCCESS</b>")
                .replace('Success', "<b>Success</b>");
            }

            if(status.toLowerCase().includes('fail') || status.toLowerCase().includes('error')) {
                return status.replace('fail', "<b>fail</b>")
                .replace('FAIL', "<b>FAIL</b>")
                .replace('Fail', "<b>Fail</b>")
                .replace('error', "<b>error</b>")
                .replace('Error', "<b>error</b>")
                .replace('ERROR', "<b>ERROR</b>");
            }
          return status;
        },
    }
}
</script>
<style>
    table {
      border-collapse: collapse;
      border-spacing: 0;
      width: 100%;
      border: 1px solid #ddd;
    }

    th, td {
      text-align: left;
      padding: 8px;
    }
    .status {
      margin-left: 20px;
    }
    tr:nth-child(even){background-color: #f2f2f2}
</style>