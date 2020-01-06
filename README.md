# BCDBIGMUL
multiplying bcd numbers
BCDBIGMUL     STMFD   SP!, {R4-R11, LR}
              MOV     R10, #0x400

              ;up     to L1, the aim is to store the positive magnitudes of the numbers stored in R0 and R1. These will be stored in the R10 stack
              SUB     R11, R3, #1
              LSL     R11, R11, #2
              LDR     R4, [R0, R11]
              CMP     R4, #0x50000000 ;checks wheather the number is already positive
              BCS     MAKE_POS1

              MOV     R8, #12
              MOV     R9, #0
L7            ADD     R8, R8, #4
              LDR     R4, [R0, R9]
              STR     R4, [R10, R8]
              ADD     R9, R9, #4
              CMP     R9, R3, LSL #2
              BNE     L7
              B       ALREADY_POS1

MAKE_POS1     MOV     R7, #0
              MOV     R9, #1
L8            MOV     R11, #0
              MOV     R8, #0
L9            LDR     R6, [R0, R7]
              LSR     R6, R6, R8
              AND     R6, R6, #0b1111
              RSB     R6, R6, #9
              ADD     R6, R6, R9
              CMP     R6, #10
              SUBGE   R6, R6, #10
              MOVLT   R9, #0
              ORR     R11, R11, R6, LSL R8
              ADD     R8, R8, #4
              CMP     R8, #32
              BLT     L9
              ADD     R4, R7, #16
              STR     R11, [R10, R4]
              ADD     R7, R7, #4
              CMP     R7, R3, LSL #2
              BLT     L8

ALREADY_POS1  SUB     R11, R3, #1
              LSL     R11, R11, #2
              LDR     R4, [R1, R11]
              CMP     R4, #0x50000000
              BCS     MAKE_POS2

              ADD     R8, R11, #16
              MOV     R9, #0
L10           ADD     R8, R8, #4
              LDR     R4, [R1, R9]
              STR     R4, [R10, R8]
              ADD     R9, R9, #4
              CMP     R9, R3, LSL #2
              BNE     L10
              B       ALREADY_POS2

MAKE_POS2     MOV     R7, #0
              MOV     R9, #1
L11           MOV     R11, #0
              MOV     R8, #0
L12           LDR     R6, [R1, R7]
              LSR     R6, R6, R8
              AND     R6, R6, #0b1111
              RSB     R6, R6, #9
              ADD     R6, R6, R9
              CMP     R6, #10
              SUBGE   R6, R6, #10
              MOVLT   R9, #0
              ORR     R11, R11, R6, LSL R8
              ADD     R8, R8, #4
              CMP     R8, #32
              BLT     L12
              ADD     R4, R7, #16
              ADD     R4, R4, R3, LSL #2
              STR     R11, [R10, R4]
              ADD     R7, R7, #4
              CMP     R7, R3, LSL #2
              BLT     L11

ALREADY_POS2  MOV     R9, #0
              MOV     R11, #0
              STR     R11, [R10]

L1            MOV     R11, #0 ;L1 is looped when R1's next word is needed (happens K times)
              STR     R11, [R10, #4]

L2            MOV     R11, #0 ;L2 is looped when R1's next digit of a specific word is needed (happens 8 times)
              STR     R11, [R10, #8]

L3            MOV     R11, #0 ;L3 is looped when R0's next word is needed (happens K times)
              STR     R11, [R10, #12]

L4            LDR     R11, [R10] ;L4 is looped when R0's next digit of a specific word is needed (happens 8 times)
              ADD     R11, R11, #4
              ADD     R11, R11, R3
              LDR     R4, [R10, R11, LSL #2]
              LDR     R11, [R10, #4]
              LSL     R11, R11, #2
              LSR     R4, R4, R11
              AND     R4, R4, #0b1111
              LDR     R11, [R10, #8]
              ADD     R11, R11, #4
              LDR     R5, [R10, R11, LSL #2]
              LDR     R11, [R10, #12]
              LSL     R11, R11, #2
              LSR     R5, R5, R11
              AND     R5, R5, #0b1111

              MOV     R6, #0 ;R6: product of R4 and R5
              MOV     R7, #0b1
              MOV     R8, #3 ;R8: downward iteration
L_MUL         AND     R11, R7, R4, LSR R8
              CMP     R11, #1
              ADDEQ   R6, R6, R5, LSL R8
              SUBS    R8, R8, #1
              BPL     L_MUL

              MOV     R5, #0
              CMP     R6, #10
L_REDUCE      SUBGE   R6, R6, #10
              ADDGE   R5, R5, #1
              CMP     R6, #10
              BGE     L_REDUCE
              ;R5:    CARRY TERM, R6: PRODUCT TERM, eg. for 81: R5=8, R6=1

              LDR     R7, [R10]
              LDR     R11, [R10, #8]
              ADD     R7, R7, R11
              LDR     R8, [R10, #4]
              LDR     R11, [R10, #12]
              ADD     R8, R8, R11
              CMP     R8, #8
              SUBGE   R8, R8, #8
              ADDGE   R7, R7, #1 ;R7: nth word of z (0 to K-1), R8: the specific digit of the nth word of z (0 to 7), where z is the output to be stored in R2

              ADD     R6, R6, R9 ;we are now adding the previous carry term to the current digit
              CMP     R6, #10
              SUBGE   R6, R6, #10
              ADDGE   R5, R5, #1

              LDR     R4, [R2, R7, LSL #2] ;R4: previous (mid-calculation) nth word of z-term
              LSL     R8, R8, #2
              LSR     R11, R4, R8
              AND     R11, R11, #0b1111 ;R11: specific digit of z-term that is to be altered

              ADD     R11, R11, R6 ;we are now altering R11
              CMP     R11, #10
              SUBGE   R11, R11, #10
              ADDGE   R5, R5, #1
              MOV     R9, R5 ;we are now storing the next carry term in R9

              ADD     R6, R8, #4
              LSR     R5, R4, R6
              LSL     R5, R5, R6
              ORR     R5, R5, R11, LSL R8
              RSB     R6, R8, #32
              LSL     R11, R4, R6
              ORR     R5, R5, R11, LSR R6
              STR     R5, [R2, R7, LSL #2]

              ;L4_END:
              LDR     R11, [R10, #12]
              ADD     R11, R11, #1
              CMP     R11, #8
              STRNE   R11, [R10, #12]
              BNE     L4

              ;L3_END:
              LDR     R11, [R10, #8]
              ADD     R11, R11, #1
              CMP     R11, R3
              STRNE   R11, [R10, #8]
              BNE     L3

              LDR     R7, [R10] ;the next 3 pieces of code will account for the final carry term. This must be done everytime the entire x-term is multiplied by a single digit of y.
              LDR     R11, [R10, #8]
              ADD     R7, R7, R11
              LDR     R8, [R10, #4]
              LDR     R11, [R10, #12]
              ADD     R8, R8, R11

L_FINAL_CARRY CMP     R8, #8
              SUBGE   R8, R8, #8
              ADDGE   R7, R7, #1
              LDR     R4, [R2, R7, LSL #2]
              LSL     R8, R8, #2
              LSR     R11, R4, R8
              AND     R11, R11, #0b1111
              ADD     R11, R11, R9
              CMP     R11, #10
              BLT     L2_END

              SUB     R11, R11, #10
              ADD     R6, R8, #4
              LSR     R5, R4, R6
              LSL     R5, R5, R6
              ORR     R5, R5, R11, LSL R8
              RSB     R6, R8, #32
              LSL     R11, R4, R6
              ORR     R5, R5, R11, LSR R6
              STR     R5, [R2, R7, LSL #2]
              MOV     R9, #1
              ADD     R8, R8, #1
              B       L_FINAL_CARRY

L2_END        LDR     R11, [R10, #4]
              ADD     R11, R11, #1
              CMP     R11, #8
              STRNE   R11, [R10, #4]
              BNE     L2

              ;L1_END:
              LDR     R11, [R10]
              ADD     R11, R11, #1
              CMP     R11, R3
              STRNE   R11, [R10]
              BNE     L1

              ;this   section checks if the answer should be positive or negative:
              SUB     R11, R3, #1
              LDR     R4, [R1, R11, LSL #2]
              LDR     R5, [R0, R11, LSL #2]
              CMP     R4, #0x50000000
              ADDCS   R11, R11, #1
              CMP     R5, #0x50000000
              ADDCS   R11, R11, #1
              CMP     R11, R3
              BNE     END_LINE ;if the answer is supposed ot be positive, we simply skip to the end

              ;this   section multiplies the answer by -1 if necessary:
              MOV     R6, #1
              MOV     R7, #0 ;iteration up to 2K
L5            MOV     R9, #0
              LDR     R4, [R2, R7, LSL #2]
              MOV     R8, #0 ;iteration for digit (up to 8x4=32) for a given word
L6            LSR     R5, R4, R8
              AND     R5, R5, #0b1111
              RSB     R5, R5, #9
              CMP     R6, #1
              BNE     SKIP
              ADD     R5, R5, R6
              CMP     R5, #10
              SUBGE   R5, R5, #10
              MOVLT   R6, #0
SKIP          ORR     R9, R9, R5, LSL R8
              ADD     R8, R8, #4
              CMP     R8, #32
              BNE     L6
              STR     R9, [R2, R7, LSL #2]
              ADD     R7, R7, #1
              CMP     R7, R3, LSL #1
              BNE     L5

END_LINE      LDMFD   SP!, {R4-R11, PC}
