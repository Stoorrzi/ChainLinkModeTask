# ChainLinkModeTask
Template Code to fulfill the ChainLink Mode-Task


type = "directrequest"
schemaVersion = 1
name = "JOB_NAME"
contractAddress = "YOUR_CONTRACT_ADDRESS"
maxTaskDuration = "0s"
observationSource = """
    decode_log   [type="ethabidecodelog"
                  abi="OracleRequest(bytes32 indexed specId, address requester, bytes32 requestId, uint256 payment, address callbackAddr, bytes4 callbackFunctionId, uint256 cancelExpiration, uint256 dataVersion, bytes data)"
                  data="$(jobRun.logData)"
                  topics="$(jobRun.logTopics)"]

    decode_cbor  [type="cborparse" data="$(decode_log.data)"]
    fetch_1      [type=bridge name="bridgeTwo" requestData="{\\"id\\": $(jobSpec.externalJobID), \\"data\\": { \\"txh\\": $(decode_cbor.txh)}}"]
    parse_1      [type=jsonparse path="result" data="$(fetch_1)"]
    fetch_2        [type=bridge name="Test1" requestData="{\\"id\\": $(jobSpec.externalJobID), \\"data\\": { \\"txh\\": $(decode_cbor.txh)}}"]
    parse_2        [type=jsonparse path="result" data="$(fetch_2)"]
    fetch_3        [type=bridge name="Test2" requestData="{\\"id\\": $(jobSpec.externalJobID), \\"data\\": { \\"txh\\": $(decode_cbor.txh)}}"]
    parse_3        [type=jsonparse path="result" data="$(fetch_3)"]
    fetch_4        [type=bridge name="Test3" requestData="{\\"id\\": $(jobSpec.externalJobID), \\"data\\": { \\"txh\\": $(decode_cbor.txh)}}"]
    parse_4        [type=jsonparse path="result" data="$(fetch_4)"]
    fetch_5        [type=bridge name="Test4" requestData="{\\"id\\": $(jobSpec.externalJobID), \\"data\\": { \\"txh\\": $(decode_cbor.txh)}}"]
    parse_5        [type=jsonparse path="result" data="$(fetch_5)"]
    my_median_task [type="median"
                values=<[ $(parse_1), $(parse_2), $(parse_3), $(parse_4), $(parse_5) ]>
                allowedFaults=2]
    encode_large [type="ethabiencode"
                abi="(bytes _data)"
                data="{\\"_data\": $(my_median_task)}"]
    encode_tx  [type="ethabiencode"
                abi="fulfillOracleRequest2(bytes32 requestId, uint256 payment, address callbackAddress, bytes4 callbackFunctionId, uint256 expiration, bytes calldata data)"
                data="{\"requestId\": $(decode_log.requestId), \"payment\":   $(decode_log.payment), \"callbackAddress\": $(decode_log.callbackAddr), \"callbackFunctionId\": $(decode_log.callbackFunctionId), \"expiration\": $(decode_log.cancelExpiration), \"data\": $(encode_large)}"
                ]

    submit_tx    [type="ethtx" to="YOUR_CONTRACT_ADDRESS" data="$(encode_tx)"]

    decode_log -> decode_cbor -> fetch_1 -> parse_1 -> fetch_2 -> parse_2 -> fetch_3 -> parse_3 -> fetch_4 -> parse_4 -> fetch_5 -> parse_5 -> my_median_task -> encode_large -> encode_tx -> submit_tx
"""
