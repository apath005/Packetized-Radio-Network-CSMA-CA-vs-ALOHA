# 3-Node-Packetized-Radio-Network-CSMA-CA
This model is intended to simulate a 3- node PHY / MAC network. The goal is to find the mechanisms related to CSMA / CA protocol within our generated source code.
For the purpose of this simulation, we are only concerned with primarily the MAC layer and transmission of data frames.
For CSMA/CA MAC , the medium must be quiet for the period of DIFS before transmitting, then the node starts the random Contention Window (CW) period. At the end of the CW, if the medium is still quiet, the node starts to transmit a Data frame; otherwise it waits for another quiet DIFS period. When the transmitting node hears an Ack frame within the SIFS period, it will transmit the next Data frame; otherwise it enters the DIFS period again.

## CSMA / CA MAC Layer traffic
![image](https://user-images.githubusercontent.com/77028776/127751222-77b0f14a-57a3-4787-a705-ffcc96cc9540.png)

- Constants 
```
62	/* Named constants for Chart: '<S14>/CSMA//CA MAC' */
63	#define Packetized_Radio_3Node_IN_AckOn_f ((uint8_T)8U)
64	#define Packetized_Radio_3Node_IN_AckWait_o ((uint8_T)1U)
65	#define Packetized_Radio_3Node_IN_Backoff_i ((uint8_T)6U)
66	#define Packetized_Radio_3Node_IN_IFS_Time ((uint8_T)7U)
67	#define Packetized_Radio_3Node_IN_NO_ACTIVE_CHILD_g ((uint8_T)0U)
68	#define Packetized_Radio_3Node_IN_Reset1_h ((uint8_T)3U)
69	#define Packetized_Radio_3Node_IN_Reset_e ((uint8_T)2U)
70	#define Packetized_Radio_3Node_IN_SIFS_Time ((uint8_T)9U)
71	#define Packetized_Radio_3Node_IN_StandBy_i ((uint8_T)10U)
72	#define Packetized_Radio_3Node_IN_TurnOnTxSw_l ((uint8_T)4U)
73	#define Packetized_Radio_3Node_IN_TxOn_c ((uint8_T)5U)
```
## The Logical Link Layer
![image](https://user-images.githubusercontent.com/77028776/127751910-1406b507-631b-4d2e-bcd7-104454e18766.png)

This process is responsible for injecting and transmitting data packets to the transceiver and uses a wait time and backoff timer to count down the packet inter-arrival time until the next packet arrives.

```
/* Chart: '<S11>/Link Logic Control ' */
/* Gateway: Radio_Node1/Link Logic Control  */
/* During: Radio_Node1/Link Logic Control  */
	  if (localDW->is_active_c17_Packet_Radio_Network_TxRx_lib == 0U) {
	    /* Entry: Radio_Node1/Link Logic Control  */
	    localDW->is_active_c17_Packet_Radio_Network_TxRx_lib = 1U;
	
	    /* Entry Internal: Radio_Node1/Link Logic Control  */
	    /* Transition: '<S12>:6' */
	    localDW->TxFlCntMax = 100.0;
	    localB->TxLength = 92.0;
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
3211	}
```

## CSMA / CA MAC Layer
The MAC Layer responsible for the CSMA / CA protocol is of importance and the process of data acknowledgement and the use of a backoff timer are considered to be key variables central to the CSMA / CA mechanism.
![image](https://user-images.githubusercontent.com/77028776/127754588-8f96330c-a9c1-4408-9012-54346affe7d7.png)

## Functions and Definitions (CSMA / CA)
```
/* Function for Chart: '<S14>/CSMA//CA MAC' */
5787	static void
5788	  Packetized_Radio_3Node_enter_internal_c22_Packet_Radio_Network_TxRx_lib(real_T
5789	  rtp_pseed, B_CSMACAMAC_Packetized_Radio_3Node_T *localB,
5790	  DW_CSMACAMAC_Packetized_Radio_3Node_T *localDW)
5791	{
5792	  int32_T r;
5793	  int32_T t;
5794	  uint32_T seed;
5795	
5796	  /* Entry Internal: Radio_Node1/Subsystem/CSMA//CA MAC */
5797	  /* Transition: '<S70>:321' */
5798	  localDW->AckLength = 25.0;
5799	  localDW->SLOT = 25.0;
5800	  localDW->DIFS = 62.0;
5801	  localDW->SIFS = 12.0;
5802	  localDW->EIFS = 100.0;
5803	  localDW->TP = 2.0;
5804	
5805	  /* Chart: '<S14>/CSMA//CA MAC' */
5806	  seed = (uint32_T)rtp_pseed;
5807	  if (localDW->method == 7U) {
5808	    /* Chart: '<S14>/CSMA//CA MAC' */
5809	    if ((uint32_T)rtp_pseed == 0U) {
5810	      seed = 5489U;
5811	    }
5812	
5813	    localDW->state_b[0] = seed;
5814	    for (r = 0; r < 623; r++) {
5815	      seed = ((seed >> 30U ^ seed) * 1812433253U + r) + 1U;
5816	      localDW->state_b[r + 1] = seed;
5817	    }
5818	
5819	    localDW->state_b[624] = 624U;
5820	  } else if (localDW->method == 5U) {
5821	    localDW->state_g[0] = 362436069U;
5822	
5823	    /* Chart: '<S14>/CSMA//CA MAC' */
5824	    localDW->state_g[1] = (uint32_T)rtp_pseed;
5825	    if (localDW->state_g[1] == 0U) {
5826	      localDW->state_g[1] = 521288629U;
5827	    }
5828	  } else {
5829	    /* Chart: '<S14>/CSMA//CA MAC' */
5830	    r = (int32_T)((uint32_T)rtp_pseed >> 16U);
5831	    t = (int32_T)((uint32_T)rtp_pseed & 32768U);
5832	    seed = (((((uint32_T)rtp_pseed - ((uint32_T)r << 16U)) - t) << 16U) + t) + r;
5833	    if (seed < 1U) {
5834	      seed = 1144108930U;
5835	    } else if (seed > 2147483646U) {
5836	      seed = 2147483646U;
5837	    }
5838	
5839	    localDW->state = seed;
5840	  }
5841	
5842	  localDW->CWmax = 31.0;
5843	  localDW->CWmin = 7.0;
5844	  localB->TxDataFail = 0.0;
5845	  localB->TxDataDone = 0.0;
5846	  localDW->SSCR = 10.0;
5847	  localB->Retry = 0.0;
5848	
5849	  /* Transition: '<S70>:373' */
5850	  localDW->CW = localDW->CWmin;
5851	  localB->BOTime = 0.0;
5852	  localDW->is_c22_Packet_Radio_Network_TxRx_lib =
5853	    Packetized_Radio_3Node_IN_StandBy_i;
5854	}
5855	
5856	/* Function for Chart: '<S14>/CSMA//CA MAC' */
5857	static real_T Packetized_Radio_3Node_eml_rand_mt19937ar_stateful_g
5858	  (DW_CSMACAMAC_Packetized_Radio_3Node_T *localDW)
5859	{
5860	  real_T r;
5861	  int32_T exitg1;
5862	  int32_T k;
5863	  int32_T kk;
5864	  uint32_T u[2];
5865	  uint32_T mti;
5866	  uint32_T y;
5867	  boolean_T b_isvalid;
5868	  boolean_T exitg2;
```
## Acknowledgement Frames
Source code:
```
6191	  /* Chart: '<S14>/CSMA//CA MAC' */
6192	  /* Gateway: Radio_Node1/Subsystem/CSMA//CA MAC */
6193	  /* During: Radio_Node1/Subsystem/CSMA//CA MAC */
6194	  if (localDW->is_active_c22_Packet_Radio_Network_TxRx_lib == 0U) {
6195	    /* Entry: Radio_Node1/Subsystem/CSMA//CA MAC */
6196	    localDW->is_active_c22_Packet_Radio_Network_TxRx_lib = 1U;
6197	    Packetized_Radio_3Node_enter_internal_c22_Packet_Radio_Network_TxRx_lib
6198	      (rtp_pseed, localB, localDW);
6199	  } else {
6200	    guard1 = false;
6201	    switch (localDW->is_c22_Packet_Radio_Network_TxRx_lib) {
6202	     case Packetized_Radio_3Node_IN_AckWait_o:
6203	      /* During 'AckWait': '<S70>:315' */
6204	      if (rtu_AckOk && (localB->AckWaitTime > 0.0)) {
6205	        /* Transition: '<S70>:322' */
6206	        localB->TxDataDone = 1.0;
6207	        localB->Retry = 0.0;
6208	        localDW->CW = localDW->CWmin;
6209	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6210	          Packetized_Radio_3Node_IN_Reset1_h;
6211	
6212	        /* Entry 'Reset1': '<S70>:368' */
6213	      } else if (localB->AckWaitTime == 0.0) {
6214	        /* Transition: '<S70>:331' */
6215	        if (localB->Retry < localDW->SSCR) {
6216	          /* Transition: '<S70>:318' */
6217	          localB->Retry++;
```
```
case Packetized_Radio_3Node_IN_AckOn_f:
6437	      /* During 'AckOn': '<S70>:397' */
6438	      if (localDW->TxAckTime == 0.0) {
6439	        /* Transition: '<S70>:401' */
6440	        localB->TxAckOn = 0.0;
6441	        if (localB->BOTime > 0.0) {
6442	          /* Transition: '<S70>:433' */
6443	          /* Transition: '<S70>:448' */
6444	          localDW->IFS = localDW->DIFS;
6445	          localDW->SigSt = 0.0;
6446	          localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6447	            Packetized_Radio_3Node_IN_IFS_Time;
6448	        } else {
6449	          /* Transition: '<S70>:435' */
6450	          localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6451	            Packetized_Radio_3Node_IN_StandBy_i;
6452	        }
6453	      } else {
6454	        /* Transition: '<S70>:403' */
6455	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6456	          Packetized_Radio_3Node_IN_AckOn_f;
6457	
6458	        /* Entry 'AckOn': '<S70>:397' */
6459	        localDW->TxAckTime--;
6460	        localB->TxAckOn = 1.0;
6461	      }
6462	      break;
6463	
6464	     case Packetized_Radio_3Node_IN_SIFS_Time:
6465	      /* During 'SIFS_Time': '<S70>:391' */
6466	      if (localDW->TxAckTP <= 0.0) {
6467	        /* Transition: '<S70>:398' */
6468	        localDW->TxAckTime = localDW->AckLength;
6469	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6470	          Packetized_Radio_3Node_IN_AckOn_f;
6471	
6472	        /* Entry 'AckOn': '<S70>:397' */
6473	        localDW->TxAckTime--;
6474	        localB->TxAckOn = 1.0;
6475	      } else {
6476	        /* Transition: '<S70>:399' */
6477	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6478	          Packetized_Radio_3Node_IN_SIFS_Time;
6479	
6480	        /* Entry 'SIFS_Time': '<S70>:391' */
6481	        localDW->TxAckTP--;
6482	      }
6483	      break;
6484	
6485	     default:
6486	      /* During 'StandBy': '<S70>:320' */
6487	      if (rtu_RxDataOk) {
6488	        /* Transition: '<S70>:392' */
6489	        /* Transition: '<S70>:395' */
6490	        localDW->TxAckTP = localDW->TP;
6491	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6492	          Packetized_Radio_3Node_IN_SIFS_Time;
6493	
6494	        /* Entry 'SIFS_Time': '<S70>:391' */
6495	        localDW->TxAckTP--;
6496	      } else if (rtu_TxEn) {
6497	        /* Transition: '<S70>:453' */
6498	        /* Transition: '<S70>:448' */
6499	        localDW->IFS = localDW->DIFS;
6500	        localDW->SigSt = 0.0;
6501	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6502	          Packetized_Radio_3Node_IN_IFS_Time;
6503	      }
6504	      break;
6505	    }
6506	
6507	    if (guard1) {
6508	      if (localB->AckWaitTime > 0.0) {
6509	        /* Transition: '<S70>:323' */
6510	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6511	          Packetized_Radio_3Node_IN_AckWait_o;
6512	
6513	        /* Entry 'AckWait': '<S70>:315' */
6514	        localB->AckWaitTime--;
6515	      }
6516	    }
6517	  }
6518	
6519	  /* End of Chart: '<S14>/CSMA//CA MAC' */
6520	}
```
## Backoff / Contention Window
Source code:
```
case Packetized_Radio_3Node_IN_Backoff_i:
6349	      /* During 'Backoff': '<S70>:324' */
6350	      if (rtu_SigD) {
6351	        /* Transition: '<S70>:460' */
6352	        /* Transition: '<S70>:489' */
6353	        /* Transition: '<S70>:490' */
6354	        /* Transition: '<S70>:448' */
6355	        localDW->IFS = localDW->DIFS;
6356	        localDW->SigSt = 0.0;
6357	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6358	          Packetized_Radio_3Node_IN_IFS_Time;
6359	      } else if (localB->BOTime > 0.0) {
6360	        /* Transition: '<S70>:360' */
6361	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6362	          Packetized_Radio_3Node_IN_Backoff_i;
6363	
6364	        /* Entry 'Backoff': '<S70>:324' */
6365	        localB->BOTime--;
6366	      } else if (localB->BOTime <= 0.0) {
6367	        /* Transition: '<S70>:328' */
6368	        localDW->TxTime = rtu_TxLength;
6369	        localDW->TxDataTP = localDW->TP;
6370	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6371	          Packetized_Radio_3Node_IN_TurnOnTxSw_l;
6372	
6373	        /* Entry 'TurnOnTxSw': '<S70>:421' */
6374	        localDW->TxDataTP--;
6375	      }
6376	      break;
6377	
6378	     case Packetized_Radio_3Node_IN_IFS_Time:
6379	      /* During 'IFS_Time': '<S70>:449' */
6380	      if (localDW->IFS == 0.0) {
6381	        /* Transition: '<S70>:451' */
6382	        if (!(localB->BOTime > 0.0)) {
6383	          /* Transition: '<S70>:477' */
6384	          r = Packetized_Radio_3Node_rand_h(localDW);
6385	          localB->BOTime = floor((localDW->CW + 1.0) * r) * localDW->SLOT;
6386	          if ((!(localDW->CW == localDW->CWmax)) && (!(localB->Retry == 0.0))) {
6387	            /* Transition: '<S70>:485' */
6388	            /* Transition: '<S70>:483' */
6389	            /* Transition: '<S70>:464' */
6390	            localDW->CW = localDW->CW * 2.0 + 1.0;
6391	            r = Packetized_Radio_3Node_rand_h(localDW);
6392	            localB->BOTime = floor((localDW->CW + 1.0) * r) * localDW->SLOT;
6393	          } else {
6394	            /* Transition: '<S70>:356' */
6395	            /* Transition: '<S70>:355' */
6396	          }
6397	        } else {
6398	          /* Transition: '<S70>:476' */
6399	        }
6400	
6401	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6402	          Packetized_Radio_3Node_IN_Backoff_i;
6403	
6404	        /* Entry 'Backoff': '<S70>:324' */
6405	        localB->BOTime--;
6406	      } else if (rtu_RxDataOk) {
6407	        /* Transition: '<S70>:457' */
6408	        /* Transition: '<S70>:395' */
6409	        localDW->TxAckTP = localDW->TP;
6410	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6411	          Packetized_Radio_3Node_IN_SIFS_Time;
6412	
6413	        /* Entry 'SIFS_Time': '<S70>:391' */
6414	        localDW->TxAckTP--;
6415	      } else if (!rtu_SigD) {
6416	        /* Transition: '<S70>:450' */
6417	        localDW->IFS--;
6418	        localDW->SigSt = 1.0;
6419	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6420	          Packetized_Radio_3Node_IN_IFS_Time;
6421	      } else if (localDW->SigSt == 1.0) {
6422	        /* Transition: '<S70>:454' */
6423	        /* Transition: '<S70>:448' */
6424	        localDW->IFS = localDW->DIFS;
6425	        localDW->SigSt = 0.0;
6426	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6427	          Packetized_Radio_3Node_IN_IFS_Time;
6428	      } else if (rtu_RxErr) {
6429	        /* Transition: '<S70>:465' */
6430	        localDW->IFS = localDW->EIFS;
6431	        localDW->is_c22_Packet_Radio_Network_TxRx_lib =
6432	          Packetized_Radio_3Node_IN_IFS_Time;
6433	      }
6434	      break;
```
