<script>
    import { onMount } from 'svelte';
    import axios from "axios";
    import { env } from '$env/dynamic/public';
    import { ethers } from "ethers";
	import project_ids from '../projects.json';
	import { get } from 'svelte/store';
	export let API_PREFIX = env.PUBLIC_API_PREFIX;
    //export let NETWORK = env.PUBLIC_NETWORK || "Ethereum Mainnet";
    export let EXPLORER_PREFIX = env.PUBLIC_EXPLORER_PREFIX || "https://etherscan.io";
    export let RPC = env.PUBLIC_RPC;
    console.log('got RPC', RPC);

    let provider = new ethers.JsonRpcProvider(RPC);
    const ABI=[
	{
		"inputs": [
			{
				"internalType": "uint8",
				"name": "epochSize",
				"type": "uint8"
			},
			{
				"internalType": "uint256",
				"name": "sourceChainId",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "sourceChainBlockTime",
				"type": "uint256"
			},
			{
				"internalType": "bool",
				"name": "useBlockNumberAsEpochId",
				"type": "bool"
			}
		],
		"stateMutability": "nonpayable",
		"type": "constructor"
	},
	{
		"inputs": [],
		"name": "InvalidShortString",
		"type": "error"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "str",
				"type": "string"
			}
		],
		"name": "StringTooLong",
		"type": "error"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "address",
				"name": "snapshotterAddr",
				"type": "address"
			},
			{
				"indexed": false,
				"internalType": "string",
				"name": "snapshotCid",
				"type": "string"
			},
			{
				"indexed": true,
				"internalType": "uint256",
				"name": "epochId",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "string",
				"name": "projectId",
				"type": "string"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "timestamp",
				"type": "uint256"
			}
		],
		"name": "DelayedSnapshotSubmitted",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [],
		"name": "EIP712DomainChanged",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "uint256",
				"name": "epochId",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "begin",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "end",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "timestamp",
				"type": "uint256"
			}
		],
		"name": "EpochReleased",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "address",
				"name": "previousOwner",
				"type": "address"
			},
			{
				"indexed": true,
				"internalType": "address",
				"name": "newOwner",
				"type": "address"
			}
		],
		"name": "OwnershipTransferred",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "string",
				"name": "projectId",
				"type": "string"
			},
			{
				"indexed": false,
				"internalType": "bool",
				"name": "allowed",
				"type": "bool"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "enableEpochId",
				"type": "uint256"
			}
		],
		"name": "ProjectsUpdated",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": true,
				"internalType": "uint256",
				"name": "epochId",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "epochEnd",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "string",
				"name": "projectId",
				"type": "string"
			},
			{
				"indexed": false,
				"internalType": "string",
				"name": "snapshotCid",
				"type": "string"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "timestamp",
				"type": "uint256"
			}
		],
		"name": "SnapshotFinalized",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "address",
				"name": "snapshotterAddr",
				"type": "address"
			},
			{
				"indexed": false,
				"internalType": "string",
				"name": "snapshotCid",
				"type": "string"
			},
			{
				"indexed": true,
				"internalType": "uint256",
				"name": "epochId",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "string",
				"name": "projectId",
				"type": "string"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "timestamp",
				"type": "uint256"
			}
		],
		"name": "SnapshotSubmitted",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "address",
				"name": "validatorAddress",
				"type": "address"
			},
			{
				"indexed": false,
				"internalType": "bool",
				"name": "allowed",
				"type": "bool"
			}
		],
		"name": "ValidatorsUpdated",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "address",
				"name": "snapshotterAddress",
				"type": "address"
			},
			{
				"indexed": false,
				"internalType": "bool",
				"name": "allowed",
				"type": "bool"
			}
		],
		"name": "allSnapshottersUpdated",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "address",
				"name": "snapshotterAddress",
				"type": "address"
			},
			{
				"indexed": false,
				"internalType": "bool",
				"name": "allowed",
				"type": "bool"
			}
		],
		"name": "masterSnapshottersUpdated",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "string",
				"name": "projectId",
				"type": "string"
			},
			{
				"indexed": false,
				"internalType": "bool",
				"name": "allowed",
				"type": "bool"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "enableEpochId",
				"type": "uint256"
			}
		],
		"name": "pretestProjectsUpdated",
		"type": "event"
	},
	{
		"inputs": [],
		"name": "EPOCH_SIZE",
		"outputs": [
			{
				"internalType": "uint8",
				"name": "",
				"type": "uint8"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "SOURCE_CHAIN_BLOCK_TIME",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "SOURCE_CHAIN_ID",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "USE_BLOCK_NUMBER_AS_EPOCH_ID",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"name": "allProjects",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"name": "allSnapshotters",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "projectId",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "epochId",
				"type": "uint256"
			}
		],
		"name": "checkDynamicConsensusSnapshot",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "currentEpoch",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "begin",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "end",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "epochId",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"name": "currentFinalizedSnapshot",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "eip712Domain",
		"outputs": [
			{
				"internalType": "bytes1",
				"name": "fields",
				"type": "bytes1"
			},
			{
				"internalType": "string",
				"name": "name",
				"type": "string"
			},
			{
				"internalType": "string",
				"name": "version",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "chainId",
				"type": "uint256"
			},
			{
				"internalType": "address",
				"name": "verifyingContract",
				"type": "address"
			},
			{
				"internalType": "bytes32",
				"name": "salt",
				"type": "bytes32"
			},
			{
				"internalType": "uint256[]",
				"name": "extensions",
				"type": "uint256[]"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"name": "epochInfo",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "timestamp",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "blocknumber",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "epochEnd",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "projectId",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "epochId",
				"type": "uint256"
			}
		],
		"name": "forceCompleteConsensusSnapshot",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "begin",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "end",
				"type": "uint256"
			}
		],
		"name": "forceSkipEpoch",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getBlock",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "projectId",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "epochId",
				"type": "uint256"
			}
		],
		"name": "getFinalizedSnapshot",
		"outputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getMasterSnapshotters",
		"outputs": [
			{
				"internalType": "address[]",
				"name": "",
				"type": "address[]"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getPretestProjects",
		"outputs": [
			{
				"internalType": "string[]",
				"name": "",
				"type": "string[]"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getProjects",
		"outputs": [
			{
				"internalType": "string[]",
				"name": "",
				"type": "string[]"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getSnapshotters",
		"outputs": [
			{
				"internalType": "address[]",
				"name": "",
				"type": "address[]"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getTotalMasterSnapshotterCount",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getTotalSnapshotterCount",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getValidators",
		"outputs": [
			{
				"internalType": "address[]",
				"name": "",
				"type": "address[]"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"name": "lastFinalizedSnapshot",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"name": "masterSnapshotters",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"name": "maxSnapshotsCid",
		"outputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"name": "maxSnapshotsCount",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "minSubmissionsForConsensus",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "owner",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"name": "pretestProjects",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"name": "projectFirstEpochId",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "bytes32",
				"name": "messageHash",
				"type": "bytes32"
			},
			{
				"internalType": "bytes",
				"name": "signature",
				"type": "bytes"
			}
		],
		"name": "recoverAddress",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "pure",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "begin",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "end",
				"type": "uint256"
			}
		],
		"name": "releaseEpoch",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "renounceOwnership",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"name": "snapshotStatus",
		"outputs": [
			{
				"internalType": "bool",
				"name": "finalized",
				"type": "bool"
			},
			{
				"internalType": "uint256",
				"name": "timestamp",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "snapshotSubmissionWindow",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			},
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"name": "snapshotsReceived",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			},
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"name": "snapshotsReceivedCount",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "snapshotCid",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "epochId",
				"type": "uint256"
			},
			{
				"internalType": "string",
				"name": "projectId",
				"type": "string"
			},
			{
				"components": [
					{
						"internalType": "uint256",
						"name": "deadline",
						"type": "uint256"
					},
					{
						"internalType": "string",
						"name": "snapshotCid",
						"type": "string"
					},
					{
						"internalType": "uint256",
						"name": "epochId",
						"type": "uint256"
					},
					{
						"internalType": "string",
						"name": "projectId",
						"type": "string"
					}
				],
				"internalType": "struct PowerloomProtocolState.Request",
				"name": "request",
				"type": "tuple"
			},
			{
				"internalType": "bytes",
				"name": "signature",
				"type": "bytes"
			}
		],
		"name": "submitSnapshot",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "newOwner",
				"type": "address"
			}
		],
		"name": "transferOwnership",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string[]",
				"name": "_projects",
				"type": "string[]"
			},
			{
				"internalType": "bool[]",
				"name": "_status",
				"type": "bool[]"
			}
		],
		"name": "updateAllProjects",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address[]",
				"name": "_snapshotters",
				"type": "address[]"
			},
			{
				"internalType": "bool[]",
				"name": "_status",
				"type": "bool[]"
			}
		],
		"name": "updateAllSnapshotters",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address[]",
				"name": "_snapshotters",
				"type": "address[]"
			},
			{
				"internalType": "bool[]",
				"name": "_status",
				"type": "bool[]"
			}
		],
		"name": "updateMasterSnapshotters",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "_minSubmissionsForConsensus",
				"type": "uint256"
			}
		],
		"name": "updateMinSnapshottersForConsensus",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string[]",
				"name": "_projects",
				"type": "string[]"
			},
			{
				"internalType": "bool[]",
				"name": "_status",
				"type": "bool[]"
			}
		],
		"name": "updatePretestProjects",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "newsnapshotSubmissionWindow",
				"type": "uint256"
			}
		],
		"name": "updateSnapshotSubmissionWindow",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "address[]",
				"name": "_validators",
				"type": "address[]"
			},
			{
				"internalType": "bool[]",
				"name": "_status",
				"type": "bool[]"
			}
		],
		"name": "updateValidators",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "snapshotCid",
				"type": "string"
			},
			{
				"internalType": "uint256",
				"name": "epochId",
				"type": "uint256"
			},
			{
				"internalType": "string",
				"name": "projectId",
				"type": "string"
			},
			{
				"components": [
					{
						"internalType": "uint256",
						"name": "deadline",
						"type": "uint256"
					},
					{
						"internalType": "string",
						"name": "snapshotCid",
						"type": "string"
					},
					{
						"internalType": "uint256",
						"name": "epochId",
						"type": "uint256"
					},
					{
						"internalType": "string",
						"name": "projectId",
						"type": "string"
					}
				],
				"internalType": "struct PowerloomProtocolState.Request",
				"name": "request",
				"type": "tuple"
			},
			{
				"internalType": "bytes",
				"name": "signature",
				"type": "bytes"
			},
			{
				"internalType": "address",
				"name": "signer",
				"type": "address"
			}
		],
		"name": "verify",
		"outputs": [
			{
				"internalType": "bool",
				"name": "",
				"type": "bool"
			}
		],
		"stateMutability": "view",
		"type": "function"
	}
];

    const contract = new ethers.Contract(env.PUBLIC_STATE_CONTRACT, ABI, provider);

	// const liteModePairs = 
	// console.log('liteModePairs', liteModePairs);

    let projects = [];
    let currentEpoch = "";
	let currentEpochId = "";
    let allSnapshotters= [];
	let allMasterSnapshotters= [];
	let search = "";
	let finalizedProjects;
	let epochs;
	let prevSearch = "";
	let data = {};
	let API = "http://localhost:9030"

	$: {
		if (search != prevSearch){
			localStorage.setItem("search_term", search);
			prevSearch = search;
		}
	}
	async function getFinalizedProjects(epochId){
		let response;
		try {
			response = await axios.post(API+`/getFinalizedProjects`, {epochId: epochId});
			console.log('got Finalized Projects', response.data);
			if (response.data) {
				return new Set(response.data.finalized_projects);
			} else {
				throw new Error(JSON.stringify(response.data));
			}
		}
		catch (e){
			console.error('Unable to get finalized projects', e);
		}
	}

	async function getSubmittedProjects(epochId, snapshotterAddr){
		let response;
		try {
			response = await axios.post(API+`/getSubmittedProjects`, {epochId: epochId, snapshotterAddr: snapshotterAddr});
			console.log('got Submitted Projects', response.data);
			if (response.data) {
				return response.data.submitted_projects;
			} else {
				throw new Error(JSON.stringify(response.data));
			}
		}
		catch (e){
			console.error('Unable to get submitted projects', e);
		}
	}



    onMount(async () => {
      search = localStorage.getItem("search_term");
	  console.log('search', search);
      currentEpoch = Object.assign({}, await contract.currentEpoch());
      currentEpochId = Number(currentEpoch[2]);
	  currentEpoch = Number(currentEpoch[1]);
      //console.warn('current epoch', currentEpoch);
      allSnapshotters = Object.values(Object.assign({}, await contract.getSnapshotters()));
	  allMasterSnapshotters = Object.values(Object.assign({}, await contract.getMasterSnapshotters()));

	  allSnapshotters = allSnapshotters.concat(allMasterSnapshotters);
	  let getProjects = project_ids["project_ids"];

	  const projectSnapshotters = Object.values(Object.assign({}, allSnapshotters));
	  console.log('current getProjects', getProjects);
      for (let i=0; i<getProjects.length; i++){
        let proj = {
              id: getProjects[i],
              snapshotters: projectSnapshotters
          }
          projects = [...projects, proj];
      }

	  epochs = [currentEpochId, currentEpochId-1, currentEpochId-2, currentEpochId-3, currentEpochId-4];
	  console.log('epochs', epochs);

	  for (let i=0; i<epochs.length; i++){
		let finalizedProjects = await getFinalizedProjects(epochs[i]);
		let submittedProjects = await getSubmittedProjects(epochs[i], search);
		data[epochs[i]] = {finalizedProjects: finalizedProjects, submittedProjects: submittedProjects};
	  }
	  console.log('data', data);

    });
</script>
<div>
    <h3 class="text-lg font-medium leading-6 text-gray-900">Onchain consensus Stats</h3>
    <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
      <div class="overflow-hidden rounded-lg bg-white px-4 py-5 shadow sm:p-6">
        <dt class="truncate text-sm font-medium text-gray-500">Total Projects</dt>
        <dd class="mt-1 text-3xl font-semibold tracking-tight text-gray-900">{projects.length}</dd>
      </div>
  
      <div class="overflow-hidden rounded-lg bg-white px-4 py-5 shadow sm:p-6">
        <dt class="truncate text-sm font-medium text-gray-500">Current Epoch</dt>
        <dd class="mt-1 text-3xl font-semibold tracking-tight text-gray-900"><a href="{EXPLORER_PREFIX}/block/{currentEpoch}" target="_blank" rel="noreferrer noopener">{currentEpoch}</a></dd>
      </div>
  
      <div class="overflow-hidden rounded-lg bg-white px-4 py-5 shadow sm:p-6">
        <dt class="truncate text-sm font-medium text-gray-500">Total Snapshotters</dt>
        <dd class="mt-1 text-3xl font-semibold tracking-tight text-gray-900">{allSnapshotters.length}</dd>
      </div>
    </dl>
</div>
<div class="overflow-hidden bg-white shadow sm:rounded-md">
	<div class="relative mt-2 flex items-center">
		<input type="text" name="search" id="search" bind:value={search} placeholder="Search snapshotters" class="block w-full rounded-md border-0 py-1.5 pr-14 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6">
		<div class="absolute inset-y-0 right-0 flex py-1.5 pr-1.5" on:click={() => {search="";}}>
		  <kbd class="inline-flex items-center rounded border border-gray-200 px-1 font-sans text-xs text-gray-400">⌫</kbd>
		</div>
	  </div>
</div>
{#if search !=  ""}
	{#each Object.entries(data) as [key, value]}
	<div class="overflow-hidden bg-white shadow sm:rounded-md">
		<div class="overflow-hidden rounded-lg bg-white px-4 py-5 shadow sm:p-6">
			<dt class="truncate text-sm font-medium text-gray-500">Epoch</dt>
			<dd class="mt-1 text-3xl font-semibold tracking-tight text-gray-900">{key}</dd>
		</div>
		<ul role="list" class="divide-y divide-gray-200 mb-4">
			{#each value.submittedProjects as project}
			<li>
				<a href="/projects/{project}" class="block hover:bg-gray-50">
				  <div class="flex items-center px-4 py-4 sm:px-6">
					<div class="min-w-0 flex-1 sm:flex sm:items-center sm:justify-between">
					  <div class="truncate">
						<div class="flex text-sm">
						  <p class="truncate font-medium text-indigo-600">{project.split(":")[0]} for {project.split(":")[1]}</p>
						  <p class="ml-1 flex-shrink-0 font-normal text-gray-500">in {project.split(":")[2]}</p>
						</div>
						
					  </div>
					  <div class="mt-4 flex-shrink-0 sm:mt-0 sm:ml-5">
						<div class="flex -space-x-1 overflow-hidden">
						  {#if value.finalizedProjects.has(project)}
						  <span class="inline-block h-6 w-6 rounded-full bg-green-100">
							<svg class="h-6 w-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
							  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
							</svg>
						  </span>
						  {/if}
						  {#if !value.finalizedProjects.has(project)}
						  <span class="inline-block h-6 w-6 rounded-full bg-red-100">
							<svg class="h-6 w-6 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
							  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
							</svg>
						  </span>
						  {/if}
						</div>
					  </div>
					</div>
					<div class="ml-5 flex-shrink-0">
					  <!-- Heroicon name: mini/chevron-right -->
					  <svg class="h-5 w-5 text-gray-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
						<path fill-rule="evenodd" d="M7.21 14.77a.75.75 0 01.02-1.06L11.168 10 7.23 6.29a.75.75 0 111.04-1.08l4.5 4.25a.75.75 0 010 1.08l-4.5 4.25a.75.75 0 01-1.06-.02z" clip-rule="evenodd" />
					  </svg>
					</div>
				  </div>
				</a>
			  </li>
			{/each}
		</ul>
	
	</div>
	{/each}
{/if}

{#if search ==  ""}
<h3 class="text-lg font-medium leading-6 text-gray-900">Please Enter your snapshotter Address</h3>
{/if}