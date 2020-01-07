# BCDBIGMUL
multiplying bcd numbers

          ;       this code uses the basic method repeated addition to implement multiplication
          ;       with special care needed for the carry bits and the designated signed inputs

          ;       the program is expected to implement:
          ;       a1 a2 a3 a4 a5 a6 a7 a8
          ;       b1 b2 b3 b4 b5 b6 b7 b8
          ;       X
          ;----------------------------
          ;       ... c8 intermediate result 1 = a * b8
          ;       ... c7 intermediate result 2 = a * b7
          ;...
          ;       +
          ;----------------------------
          ;       result


BCDBIGMUL STMFD   R13!, {R4-R11, LR}
          SUBS    R10, R3, #0b1
          LSLS    R10, R10, #0b10
          LDR     R5, [R0, R10]
          MOVS    R11, #1024
          CMP     R5, #1342177280
          BLHS    SUBCHECK1
          BHS     JUMP
          MOVS    R8, #0x0
          MOVS    R9, #0xC
          
LOOP6     ADDS    R9, R9, #0x4
          LDR     R5, [R0, R8]
          STR     R5, [R11, R9]
          ADDS    R8, R8, #0x4
          LSL     R3, R3, #0b10
          CMP     R8, R3
          LSR     R3, R3, #0b10
          BNE     LOOP6
          
JUMP      SUBS    R10, R3, #0b1
          LSLS    R10, R10, #0b10
          LDR     R5, [R1, R10]
          CMP     R5, #1342177280
          BLHS    SUBCHECK2
          BHS     JUMP1
          MOV     R8, #0x0
          ADDS    R9, R10, #0x10
          
LOOP3     ADDS    R9, R9, #0x4
          LDR     R5, [R1, R8]
          STR     R5, [R11, R9]
          ADDS    R8, R8, #0x4
          LSL     R3, R3, #0b10
          CMP     R8, R3
          LSR     R3, R3, #0b10
          BNE     LOOP3
          
JUMP1     MOV     R8, #0x0
          MOV     R10, #0x0
          STR     R10, [R11]
          
MAIN1     MOV     R10, #0x0
          STR     R10, [R11, #0b100]
          
MAIN2     MOVS    R10, #0x0
          STR     R10, [R11, #0b1000]
          
MAIN3     MOVS    R10, #0x0
          STR     R10, [R11, #0xC]
          
MAIN4     LDR     R10, [R11]
          ADDS    R10, R10, #0b100
          ADDS    R10, R10, R3
          LDR     R5, [R11, R10, LSL #0b10]
          LDR     R10, [R11, #0x4]
          LSLS    R10, R10, #0b10
          LSRS    R5, R5, R10
          ANDS    R5, R5, #0xF
          LDR     R10, [R11, #0x8]
          ADDS    R10, R10, #0x4
          LDR     R4, [R11, R10, LSL #0b10]
          LDR     R10, [R11, #0xC]
          LSLS    R10, R10, #0b10
          LSRS    R4, R4, R10
          ANDS    R4, R4, #0xF
          MOV     R7, #0x0
          MOVS    R9, #0x3
          MOV     R6, #1
TIMESLOOP ANDS    R10, R6, R5, LSR R9
          CMP     R10, #0b1
          ADDEQ   R7, R7, R4, LSL R9
          SUB     R9, R9, #0b1
          CMP     R9, #0
          BGE     TIMESLOOP
          MOV     R4, #0x0
          CMP     R7, #0xA
          
BCDLOOP   SUBGE   R7, R7, #0xA
          ADDGE   R4, R4, #0b1
          CMP     R7, #0xA
          BGE     BCDLOOP
          LDR     R6, [R11]
          LDR     R10, [R11, #0x8]
          ADD     R6, R6, R10
          LDR     R9, [R11, #0x4]
          LDR     R10, [R11, #0xC]
          ADD     R9, R9, R10
          CMP     R9, #0x8
          SUBGE   R9, R9, #0x8
          ADDGE   R6, R6, #0b1
          ADD     R7, R7, R8
          CMP     R7, #0x9
          SUBGT   R7, R7, #0xA
          ADDGT   R4, R4, #0b1
          LDR     R5, [R2, R6, LSL #0b10]
          LSL     R9, R9, #0b10
          LSR     R10, R5, R9
          AND     R10, R10, #0xF
          ADD     R10, R10, R7
          CMP     R10, #0x9
          SUBGT   R10, R10, #0xA
          ADDGT   R4, R4, #0b1
          ADD     R7, R9, #0x4
          MOV     R8, R4
          LSR     R4, R5, R7
          LSL     R4, R4, R7
          ORR     R4, R4, R10, LSL R9
          RSB     R7, R9, #0x20
          LSL     R10, R5, R7
          ORR     R4, R4, R10, LSR R7
          LDR     R10, [R11, #0xC]
          STR     R4, [R2, R6, LSL #0b10]
          ADD     R10, R10, #0b1
          CMP     R10, #0x8
          STRNE   R10, [R11, #0xC]
          BNE     MAIN4
          LDR     R10, [R11, #0x8]
          ADD     R10, R10, #0b1
          CMP     R10, R3
          STRNE   R10, [R11, #0x8]
          BNE     MAIN3
          LDR     R10, [R11, #0x8]
          LDR     R6, [R11]
          ADD     R6, R6, R10
          LDR     R9, [R11, #0x4]
          LDR     R10, [R11, #0xC]
          ADD     R9, R9, R10
          
LOOPLASTC CMP     R9, #0x7
          SUBGT   R9, R9, #0x8
          ADDGT   R6, R6, #0b1
          LDR     R5, [R2, R6, LSL #0b10]
          LSL     R9, R9, #0b10
          LSR     R10, R5, R9
          AND     R10, R10, #0xF
          ADD     R10, R10, R8
          CMP     R10, #0xA
          BLT     LOOP11E
          SUB     R10, R10, #0xA
          ADD     R7, R9, #0x4
          LSR     R4, R5, R7
          LSL     R4, R4, R7
          ORR     R4, R4, R10, LSL R9
          RSB     R7, R9, #0x20
          LSL     R10, R5, R7
          ORR     R4, R4, R10, LSR R7
          STR     R4, [R2, R6, LSL #0b10]
          MOV     R8, #0b1
          ADD     R9, R9, #0b1
          B       LOOPLASTC
          
LOOP11E   LDR     R10, [R11, #0x4]
          ADD     R10, R10, #0b1
          CMP     R10, #0x8
          STRNE   R10, [R11, #0x4]
          BNE     MAIN2
          LDR     R10, [R11]
          ADDS    R10, R10, #0b1
          CMP     R10, R3
          STRNE   R10, [R11]
          BNE     MAIN1
          SUBS    R10, R3, #0b1
          LDR     R5, [R1, R10, LSL #0b10]
          LDR     R4, [R0, R10, LSL #0b10]
          CMP     R5, #1342177280
          ADDCS   R10, R10, #0b1
          CMP     R4, #1342177280
          ADDCS   R10, R10, #0b1
          CMP     R10, R3
          BNE     FINISH
          MOV     R7, #0b1
          MOVS    R6, #0x0
LOOP8     MOVS    R8, #0x0
          LDR     R5, [R2, R6, LSL #0b10]
          MOVS    R9, #0x0
LOOP7     LSRS    R4, R5, R9
          ANDS    R4, R4, #0xF
          RSBS    R4, R4, #0x9
          CMP     R7, #0b1
          BNE     JUMP2
          ADDS    R4, R4, R7
          CMP     R4, #0x9
          SUBGT   R4, R4, #0xA
          MOVLE   R7, #0x0
JUMP2     ORRS    R8, R8, R4, LSL R9
          ADDS    R9, R9, #0x4
          CMP     R9, #0x20
          BNE     LOOP7
          STR     R8, [R2, R6, LSL #0b10]
          ADDS    R6, R6, #0b1
          CMP     R6, R3, LSL #0b1
          BNE     LOOP8
FINISH    LDMFD   R13!, {R4-R11, LR}
          MOV     PC, LR
          
SUBCHECK1 MOVS    R6, #0x0
          MOVS    R8, #0b1
LOOP5     MOVS    R10, #0x0
          MOVS    R9, #0x0
LOOP4     LDR     R7, [R0, R6]
          LSRS    R7, R7, R9
          ANDS    R7, R7, #0xF
          RSBS    R7, R7, #0x9
          ADDS    R7, R7, R8
          CMP     R7, #0x9
          SUBGT   R7, R7, #0xA
          MOVLE   R8, #0x0
          ORRS    R10, R10, R7, LSL R9
          ADDS    R9, R9, #0x4
          CMP     R9, #0x1C
          BLE     LOOP4
          ADDS    R5, R6, #0x10
          STR     R10, [R11, R5]
          ADDS    R6, R6, #0x4
          CMP     R6, R3, LSL #0b10
          BLT     LOOP5
          MOV     PC, LR
          
SUBCHECK2 MOV     R6, #0x0
          MOVS    R8, #0b1
LOOP2     MOVS    R10, #0x0
          MOVS    R9, #0x0
LOOP1     LDR     R7, [R1, R6]
          LSRS    R7, R7, R9
          ANDS    R7, R7, #0xF
          RSBS    R7, R7, #0x9
          ADDS    R7, R7, R8
          CMP     R7, #0x9
          MOVLE   R8, #0x0
          SUBGT   R7, R7, #0xA
          ORRS    R10, R10, R7, LSL R9
          ADDS    R9, R9, #0x4
          CMP     R9, #0x1C
          BLE     LOOP1
          ADDS    R5, R6, #0x10
          LSL     R3, R3, #0b10
          ADDS    R5, R5, R3
          LSR     R3, R3, #0b10
          STR     R10, [R11, R5]
          ADDS    R6, R6, #0x4
          CMP     R6, R3, LSL #0b10
          BLT     LOOP2
          MOV     PC, LR
