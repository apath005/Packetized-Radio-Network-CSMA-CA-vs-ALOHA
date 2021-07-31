# 3-Node-Packetized-Radio-Network-CSMA-CA
This model is intended to simulate a 3- node PHY / MAC network. The goal is to find the mechanisms related to CSMA / CA protocol within our generated source code.
For the purpose of this simulation, we are only concerned with primarily the MAC layer and transmission of data frames.
For CSMA/CA MAC , the medium must be quiet for the period of DIFS before transmitting, then the node starts the random Contention Window (CW) period. At the end of the CW, if the medium is still quiet, the node starts to transmit a Data frame; otherwise it waits for another quiet DIFS period. When the transmitting node hears an Ack frame within the SIFS period, it will transmit the next Data frame; otherwise it enters the DIFS period again.

## CSMA / CA MAC Layer traffic
![image](https://user-images.githubusercontent.com/77028776/127751222-77b0f14a-57a3-4787-a705-ffcc96cc9540.png)

## The Logical Link Layer
![image](https://user-images.githubusercontent.com/77028776/127751910-1406b507-631b-4d2e-bcd7-104454e18766.png)

This process is responsible for injecting and transmitting data packets to the transceiver and uses a wait time and backoff timer to count down the packet inter-arrival time until the next packet arrives.
/* Chart: '<S11>/Link Logic Control ' */
3064	  /* Gateway: Radio_Node1/Link Logic Control  */
3065	  /* During: Radio_Node1/Link Logic Control  */
3066	  if (localDW->is_active_c17_Packet_Radio_Network_TxRx_lib == 0U) {
3067	    /* Entry: Radio_Node1/Link Logic Control  */
3068	    localDW->is_active_c17_Packet_Radio_Network_TxRx_lib = 1U;
3069	
3070	    /* Entry Internal: Radio_Node1/Link Logic Control  */
3071	    /* Transition: '<S12>:6' */
3072	    localDW->TxFlCntMax = 100.0;
3073	    localB->TxLength = 92.0;
3074	    if (rtp_pseed < 4.294967296E+9) {
3075	      if (rtp_pseed >= 0.0) {
3076	        seed = (uint32_T)rtp_pseed;
3077	      } else {
3078	        seed = 0U;
3079	      }
3080	    } else if (rtp_pseed >= 4.294967296E+9) {
3081	      seed = MAX_uint32_T;
3082	    } else {
3083	      seed = 0U;
3084	    }
3085	
3086	    if (localDW->method == 7U) {
3087	      if (seed == 0U) {
3088	        seed = 5489U;
3089	      }
3090	
3091	      localDW->state[0] = seed;
3092	      for (g_r = 0; g_r < 623; g_r++) {
3093	        seed = ((seed >> 30U ^ seed) * 1812433253U + g_r) + 1U;
3094	        localDW->state[g_r + 1] = seed;
3095	      }
3096	
3097	      localDW->state[624] = 624U;
3098	    } else if (localDW->method == 5U) {
3099	      localDW->state_a[0] = 362436069U;
3100	      localDW->state_a[1] = seed;
3101	      if (localDW->state_a[1] == 0U) {
3102	        localDW->state_a[1] = 521288629U;
3103	      }
3104	    } else {
3105	      g_r = (int32_T)(seed >> 16U);
3106	      t = (int32_T)(seed & 32768U);
3107	      seed = ((((seed - ((uint32_T)g_r << 16U)) - t) << 16U) + t) + g_r;
3108	      if (seed < 1U) {
3109	        seed = 1144108930U;
3110	      } else if (seed > 2147483646U) {
3111	        seed = 2147483646U;
3112	      }
3113	
3114	      localDW->state_o = seed;
3115	    }
3116	
3117	    localDW->TxFlCnt = 0.0;
3118	
3119	    /* Transition: '<S12>:11' */
3120	    c_r = Packetized_Radio_3Node_rand(localDW);
3121	    localB->TxWaitTime = floor((rtp_lambda + 1.0) * c_r);
3122	    c_r = Packetized_Radio_3Node_rand(localDW);
3123	    localB->TxMore = (floor(c_r * 4.0) + 1.0) - 1.0;
3124	    localDW->is_c17_Packet_Radio_Network_TxRx_lib =
3125	      Packetized_Radio_3Node_IN_TxBackOn;
3126	
3127	    /* Entry 'TxBackOn': '<S12>:33' */
3128	    localB->TxWaitTime--;
3129	    localB->TxEn = 0.0;
3130	  } else {
3131	    switch (localDW->is_c17_Packet_Radio_Network_TxRx_lib) {
3132	     case Packetized_Radio_3Node_IN_TxBackOn:
3133	      /* During 'TxBackOn': '<S12>:33' */
3134	      if (localB->TxWaitTime <= 0.0) {
3135	        /* Transition: '<S12>:34' */
3136	        localDW->is_c17_Packet_Radio_Network_TxRx_lib =
3137	          Packetized_Radio_3Node_IN_TxPacket;
3138	
3139	        /* Entry 'TxPacket': '<S12>:1' */
3140	        localB->TxEn = 1.0;
3141	      } else {
3142	        /* Transition: '<S12>:35' */
3143	        localDW->is_c17_Packet_Radio_Network_TxRx_lib =
3144	          Packetized_Radio_3Node_IN_TxBackOn;
3145	
3146	        /* Entry 'TxBackOn': '<S12>:33' */
3147	        localB->TxWaitTime--;
3148	        localB->TxEn = 0.0;
3149	      }
3150	      break;
3151	
3152	     case Packetized_Radio_3Node_IN_TxFailCount:
3153	      /* During 'TxFailCount': '<S12>:4' */
3154	      if (localDW->TxFlCnt == localDW->TxFlCntMax) {
3155	        /* Transition: '<S12>:8' */
3156	        localB->StopSim = 1.0;
3157	        localB->TxEn = 0.0;
3158	      } else if (localDW->TxFlCnt < localDW->TxFlCntMax) {
3159	        /* Transition: '<S12>:26' */
3160	        /* Transition: '<S12>:11' */
3161	        c_r = Packetized_Radio_3Node_rand(localDW);
3162	        localB->TxWaitTime = floor((rtp_lambda + 1.0) * c_r);
3163	        c_r = Packetized_Radio_3Node_rand(localDW);
3164	        localB->TxMore = (floor(c_r * 4.0) + 1.0) - 1.0;
3165	        localDW->is_c17_Packet_Radio_Network_TxRx_lib =
3166	          Packetized_Radio_3Node_IN_TxBackOn;
3167	
3168	        /* Entry 'TxBackOn': '<S12>:33' */
3169	        localB->TxWaitTime--;
3170	        localB->TxEn = 0.0;
3171	      }
3172	      break;
3173	
3174	     default:
3175	      /* During 'TxPacket': '<S12>:1' */
3176	      if ((rtu_TxDataDone == 1.0) && (localB->TxMore <= 0.0)) {
3177	        /* Transition: '<S12>:3' */
3178	        /* Transition: '<S12>:11' */
3179	        c_r = Packetized_Radio_3Node_rand(localDW);
3180	        localB->TxWaitTime = floor((rtp_lambda + 1.0) * c_r);
3181	        c_r = Packetized_Radio_3Node_rand(localDW);
3182	        localB->TxMore = (floor(c_r * 4.0) + 1.0) - 1.0;
3183	        localDW->is_c17_Packet_Radio_Network_TxRx_lib =
3184	          Packetized_Radio_3Node_IN_TxBackOn;
3185	
3186	        /* Entry 'TxBackOn': '<S12>:33' */
3187	        localB->TxWaitTime--;
3188	        localB->TxEn = 0.0;
3189	      } else if (rtu_TxDataFail == 1.0) {
3190	        /* Transition: '<S12>:7' */
3191	        localB->TxEn = 0.0;
3192	        localDW->is_c17_Packet_Radio_Network_TxRx_lib =
3193	          Packetized_Radio_3Node_IN_TxFailCount;
3194	
3195	        /* Entry 'TxFailCount': '<S12>:4' */
3196	        localDW->TxFlCnt++;
3197	      } else if (rtu_TxDataDone == 1.0) {
3198	        /* Transition: '<S12>:2' */
3199	        localB->TxMore--;
3200	        localDW->is_c17_Packet_Radio_Network_TxRx_lib =
3201	          Packetized_Radio_3Node_IN_TxPacket;
3202	
3203	        /* Entry 'TxPacket': '<S12>:1' */
3204	        localB->TxEn = 1.0;
3205	      }
3206	      break;
3207	    }
3208	  }
3209	
3210	  /* End of Chart: '<S11>/Link Logic Control ' */
3211	
## Acknowledgement Frames
Source code:



## Backoff / Contention Window
Source code:
