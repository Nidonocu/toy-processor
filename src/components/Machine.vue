<template>
    <div class="container">
        <div class="row">
            <div class="col-6">
                <h3>Address Space</h3>
                <div v-for="(cell, index) in memory" :key="index">
                    {{pad(index.toString(), 3, 0)}}: <strong>{{cell}}</strong>
                    <span v-if="index == pointer"><i class="ml-2 fa fa-arrow-left fa-fw"></i></span>
                </div>
            </div>
            <div class="col-6">
                <h3>State</h3>
                <div>Status: <span v-if="halted" class="text-danger">Halted</span><span v-else class="text-success">Running</span></div>
                <div>Pointer: {{pointer}}</div>
                <div>
                    Registers:
                    <ul>
                        <li>A: <strong>{{registers.a}}</strong></li>
                        <li>Z: <strong>{{registers.z}}</strong></li>
                        <li>C: <strong>{{registers.c}}</strong></li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import { program } from './Program';

export default {
    data() {
        return {
            halted: false,
            clockSpeed: 10,
            memory: [],
            pointer: 0,

            registers: {
                a: 0,
                z: 0,
                c: 0,
            },

            instructions: [
                { value: 0, ref: 'wait'},
                { value: 1, ref: 'loadValueToA'},
                { value: 2, ref: 'loadAddressToA'},
                { value: 10, ref: 'writeAToAddress'},
                { value: 20, ref: 'incrementA'},
                { value: 21, ref: 'decrementA'},
                { value: 22, ref: 'addValueToA'},
                { value: 23, ref: 'addAddressToA'},
                { value: 30, ref: 'jumpOnZero'},
                { value: 40, ref: 'compareValueToA'},
                { value: 255, ref: 'halt'},
            ]
        };
    },
    mounted() {
        for (let index = 0; index < 256; index++) {
            this.memory.push(0);
        }

        let p = program;
        
        p.forEach((element, index) => {
            this.memory.splice(index, 1, element);
        });

        setTimeout(() => {
            this.execute();
        }, this.clockSpeed);
    },
    methods: {
        pad: function (num, padlen, padchar) {
            var pad_char = typeof padchar !== 'undefined' ? padchar : '0';
            var pad = new Array(1 + padlen).join(pad_char);
            return (pad + num).slice(-pad.length);
        },

        checkPointer() {
            if (this.pointer > 255) {
                this.pointer = 0;
            }
        },

        checkAndAdvancePointer() {
            this.pointer++;
            this.checkPointer();
        },

        checkZero($result) {
            if ($result == 0) {
                this.registers.z = 1;
            } else {
                this.registers.z = 0;
            }
        },

        execute() {
            this.checkPointer();

            let instructionCode = this.memory[this.pointer];

            // Find instruction
            let instruction = this.instructions.find((i) => i.value == instructionCode);

            if (instruction == undefined) {
                console.error('Instruction Undefined');
                this.halted = true;
                return;
            }

            if (this[instruction.ref]() == 1) {
                this.halted = true;
                return;
            }

            setTimeout(() => {
                this.execute();
            }, this.clockSpeed);
        },

        wait() {
            this.pointer++;
        },

        loadValueToA() {
            this.checkAndAdvancePointer();

            this.registers.a = this.memory[this.pointer];

            this.pointer++;
        },

        loadAddressToA() {
            this.checkAndAdvancePointer();

            this.registers.a = this.memory[this.memory[this.pointer]];

            this.pointer++;
        },

        writeAToAddress() {
            this.checkAndAdvancePointer();
            
            this.memory[this.memory[this.pointer]] = this.registers.a;

            this.pointer++;
        },

        incrementA() {
            this.registers.a++;

            if (this.registers.a > 255) { 
                this.registers.a = 0;
                this.registers.c = 1;
            } else {
                this.registers.c = 0;
            }

            this.checkZero(this.registers.a);

            this.pointer++;
        },

        decrementA() {
            this.registers.a--;

            if (this.registers.a < 0) { 
                this.registers.a = 255;
            }

            this.checkZero(this.registers.a);

            this.pointer++;
        },

        addValueToA() {
            this.checkAndAdvancePointer();

            this.registers.a = this.registers.a + this.memory[this.pointer];

            if (this.registers.a > 255) {
                this.registers.a = this.registers.a - 255;
                this.registers.c = 1
            } else {
                this.registers.c = 0;
            }

            this.checkZero(this.registers.a);

            this.pointer++;
        },

        addAddressToA() {
            this.checkAndAdvancePointer();

            this.registers.a = this.registers.a + this.memory[this.memory[this.pointer]];

            if (this.registers.a > 255) {
                this.registers.a = this.registers.a - 255;
                this.registers.c = 1;
            } else {
                this.registers.c = 0;
            }

            this.checkZero(this.registers.a);

            this.pointer++;
        },

        jumpOnZero() {
            this.checkAndAdvancePointer();

            if (this.registers.z == 1) {
                this.pointer = this.memory[this.pointer];
            } else {
                this.pointer++;
            }
        },

        compareValueToA() {
            this.checkAndAdvancePointer();

            let test = this.registers.a - this.memory[this.pointer];
            
            this.checkZero(test);

            if (test > 255) {
                this.registers.c = 1;
            } else {
                this.registers.c = 0;
            }

            this.pointer++;
        },

        halt() {
            return 1;
        }
    }
}
</script>