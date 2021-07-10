---
title: Build a Decentralized Autonomous Organization (DAO) on Celo - Part 4
description: Learn how to build a fully functional DAO by writing the Smart Contract Code and then build a React Native App to communicate with the Smart Contract
---

# Build a Decentralized Autonomous Organization (DAO) on Celo - Part 4

In this tutorial, we are going to build a fully functional DAO by writing the Smart Contract Code and then build a React Native App to communicate with the Smart Contract.
This part 4 section focuses on writing the Redux code to connect the React Native App to the Smart Contract using Redux.

&nbsp;

# Prerequisite

[Redux](https://redux.js.org/introduction/getting-started)

&nbsp;

# Redux 
Install the Redux libraries using the following command:
```
yarn add redux redux-thunk redux-logger react-redux
```

```
import { applyMiddleware, combineReducers, createStore } from "redux";
import { createLogger } from "redux-logger";
import thunkMiddleware from "redux-thunk";

import { alert } from './reducers/alertReducer';
import { profile } from "./reducers/profileReducer";
import { proposal } from './reducers/proposalReducer';


const rootReducer = combineReducers({profile, proposal, alert});
const loggerMiddleware = createLogger();

export const store = createStore(
  rootReducer,
  applyMiddleware(
    thunkMiddleware,
    loggerMiddleware
  )
);
```

# Actions
## AlertActions.js
```
function request(message) {
    return {type: alertConstants.REQUEST, message};
}


function success(message) {
    return {type: alertContants.SUCCESS, message};
}


function error(message) {
    return {type: alertContants.ERROR, message};
}


function clear() {
    return {type: alertContants.CLEAR};
}
```

Contains redux action creators for actions related to toaster notifications in the dAPP. For example to display a success alert message with the text 'Wallet connection Successful' you can call `dispatch(alertActions.success('Wallet connection successful'));`.

```
function connect(params) {
  return async (dispatch )=> {
    dispatch(alertActions.clear());
    dispatch(alertActions.request("Connecting to wallet..."));
    dispatch(request())


    try {
      const goldToken = await kit.contracts.getGoldToken();
      const contractAddress = await (await contractInstance).options.address;
      const requestId = 'charlo_login';
      const dappName = 'Charlo';
      const callback = Linking.makeUrl('ProposalsPage');


      requestAccountAddress({
        requestId,
        dappName,
        callback
      });


      const response = await waitForAccountAuth(requestId);


      kit.defaultAccount = response.address;
      response.daoBalance = web3.utils.fromWei((await goldToken.balanceOf(contractAddress)).toString(), 'ether');
      response.userBalance = web3.utils.fromWei((await goldToken.balanceOf(response.address)).toString(), 'ether');
      dispatch(success(response));
      dispatch(alertActions.success("Connection successful"))
    } catch (err) {
      dispatch(alertActions.error(err.toString()));
        dispatch(failed());
    }
  }


  function request() { return { type: profileConstants.CONNECT_REQUEST} };
  function success(res) { return { type: profileConstants.CONNECT_SUCCESS, res } };
  function failed() { return { type: profileConstants.CONNECT_FAILED } };
}
```

This is the redux action creator for connecting to the wallet. This is an async function that can be called to connect to the Alfajores wallet. It is made up of sub-actions. It dispatches a request action before calling the necessary Celo library code to connect to the wallet. The request dispatch is needed to the user can be shown a notification of the next process in the wallet connection. After successfully connecting to the wallet, the default kit account is set to the address gotten from the wallet. Necessary steps were taken to get the balance of the user and DAO so that information can be displayed to the user. Also, the success action creator is dispatched with the response object to be handled by the reducer.

```
function contribute(amount) {
  return async (dispatch) => {
    dispatch(alertActions.clear());
    dispatch(alertActions.request('Contribution initiated...'));
    dispatch(request());


    try {
      const goldToken = await kit.contracts.getGoldToken();
      const contractAddress = await (await contractInstance).options.address
      const requestedAmount = web3.utils.toWei(amount, 'ether');
      const txObject = goldToken.transfer(contractAddress, requestedAmount).txo;


      const requestId = 'contribution';
      const dappName = 'Charlo';
      const callback = Linking.makeUrl('ProfilePage');


      requestTxSig(
        kit,
        [
          {
            tx: txObject,
            from: kit.defaultAccount,
            to: goldToken.contract.options.address,
            feeCurrency: FeeCurrency.cUSD
          }
        ],
        { requestId, dappName, callback }
      );


      const response = await waitForSignedTxs(requestId);
      const rawTx = response.rawTxs[0];


      // Send the signed transaction via the kit
      const tx = await kit.connection.sendSignedTransaction(rawTx);
      const receipt = await tx.waitReceipt();


      receipt.daoBalance = web3.utils.fromWei((await goldToken.balanceOf(contractAddress)).toString(), 'ether');
      receipt.userBalance = web3.utils.fromWei((await goldToken.balanceOf(kit.defaultAccount)).toString(), 'ether');


      dispatch(success(receipt));
      dispatch(alertActions.success("Contribution successful"));
    } catch (err) {
      dispatch(alertActions.error(err.toString()));
      dispatch(failed());
    }
  };


  function request() { return { type: profileConstants.CONTRIBUTION_REQUEST } };
  function success(res) { return { type: profileConstants.CONTRIBUTION_SUCCESS, res } };
  function failed() { return { type: profileConstants.CONTRIBUTION_FAILED } };
}


function payCharity(proposal) {
  return async (dispatch) => {


    if (proposal.for <= proposal.against) {
      dispatch(alertActions.error("Proposal does not have enough support"));


      return;
    }


    dispatch(alertActions.clear());
    dispatch(alertActions.request('Payment initiated...'));
    dispatch(request());


    try {
      const contractAddress = await (await contractInstance).options.address
      const txObject = await (await contractInstance).methods.payCharity(proposal.id);


      const requestId = 'pay_charity';
      const dappName = 'Charlo';
      const callback = Linking.makeUrl('ProfilePage');


      requestTxSig(
        kit,
        [
          {
            tx: txObject,
            from: kit.defaultAccount,
            to: contractAddress,
            feeCurrency: FeeCurrency.cUSD
          }
        ],
        { requestId, dappName, callback }
      );


      const response = await waitForSignedTxs(requestId);
      const rawTx = response.rawTxs[0];


      // Send the signed transaction via the kit
      const tx = await kit.connection.sendSignedTransaction(rawTx);
      const receipt = await tx.waitReceipt();


      dispatch(success(receipt));
      dispatch(alertActions.success("Payment successful"));
    } catch (err) {
      dispatch(alertActions.error(err.toString()));
      dispatch(failed());
    }
  };


  function request() { return { type: profileConstants.PAY_CHARITY_REQUEST } };
  function success(res) { return { type: profileConstants.PAY_CHARITY_SUCCESS, res } };
  function failed() { return { type: profileConstants.PAY_CHARITY_FAILED } };
}


function grantRole(amount) {
  return async (dispatch) => {
    dispatch(alertActions.clear());
    dispatch(alertActions.request('Checking for role assignment'));
    dispatch(request());


    try {
      const contractAddress = await (await contractInstance).options.address;
  
      const requestId = 'check_role';
      const dappName = 'Charlo';
      const callback = Linking.makeUrl('ProfilePage');
  
      const txObject = await (await contractInstance).methods.makeStakeholder(web3.utils.toWei(amount, 'ether'));
  
      requestTxSig(
        kit,
        [
          {
            from: kit.defaultAccount,
            to: contractAddress,
            tx: txObject,
            feeCurrency: FeeCurrency.cUSD
          }
        ],
        {requestId, dappName, callback}
      );


      const response = await waitForSignedTxs(requestId);
      const rawTx = response.rawTxs[0];
      const result = await toTxResult(kit.web3.eth.sendSignedTransaction(rawTx)).waitReceipt();


      dispatch(success(result));
      dispatch(alertActions.success('Role assignment successful'));
    } catch (err) {
      dispatch(alertActions.error(err.toString()));
      dispatch(failed());
    }
  };


  function request() { return { type: profileConstants.ROLE_REQUEST } };
  function success(res) { return { type: profileConstants.ROLE_SUCCESS, res } };
  function failed() { return { type: profileConstants.ROLE_FAILED } };
}


function getRole() {
  return async (dispatch) => {
    dispatch(alertActions.clear());
    dispatch(alertActions.request("Fetching role..."));
    dispatch(request());


    try {
      const account = kit.defaultAccount;
      let isStakeholder = await (await contractInstance).methods.isStakeholder().call({from: account});


      let isContributor = false;
      let contributed = "0";


      if (isStakeholder) {
        contributed = await (await contractInstance).methods.getStakeholderBalance().call({from: account});
      } else {
        isContributor = await (await contractInstance).methods.isContributor().call({from: account});


        if (isContributor) {
          contributed = await (await contractInstance).methods.getContributorBalance().call({from: account});
        }
      }


      dispatch(success({
        isStakeholder,
        isContributor: isStakeholder ? isStakeholder : isContributor,
        contributed: web3.utils.fromWei(contributed, 'ether')
      }));
      dispatch(alertActions.success("Role fetch successful"));
    } catch (err) {
      dispatch(alertActions.error(err.toString()));
      dispatch(failed())
    }
  }


  function request() { return { type: profileConstants.ROLE_CHECK_REQUEST}};
  function success(role) { return { type: profileConstants.ROLE_CHECK_SUCCESS, role}};
  function failed() { return { type: profileConstants.ROLE_CHECK_FAILED}};
}


function getVotes() {
  return async (dispatch) => {
    dispatch(alertActions.clear());
    dispatch(alertActions.request("Fetching votes..."));
    dispatch(request());


    try {
      const account = kit.defaultAccount;
      let votes = await (await contractInstance).methods.getStakeholderVotes().call({from: kit.defaultAccount});


      dispatch(success(votes));
      dispatch(alertActions.success("Role fetch successful"));
    } catch (err) {
      dispatch(alertActions.error(err.toString()));
      dispatch(failed())
    }
  };
  
  function request() { return { type: profileConstants.GET_VOTES_REQUEST } }
  function success(votes) { return { type: profileConstants.GET_VOTES_SUCCESS, votes } }
  function failed() { return { type: profileConstants.GET_VOTES_FAILED } }
}
```
All the other actions in this file take the same style as the one explained above.

# Reducers
## Profile Reducer

```
import { profileConstants } from '../../constants';

const initialState = {
  address: '',
  phone: '',
  daoBalance: '0',
  userBalance: '0',
  isStakeholder: false,
  isContributor: false,
  contributed: "0",
  votes: [],
  txHash: '',
  loadingPay: false,
  loading: false
};

export function profile(state = initialState, action) {
  switch (action.type) {
    case profileConstants.CONNECT_REQUEST:
    case profileConstants.CONTRIBUTION_REQUEST:
    case profileConstants.ROLE_REQUEST:
    case profileConstants.ROLE_CHECK_REQUEST:
    case profileConstants.GET_VOTES_REQUEST:
      return {
        ...state,
        txHash: '',
        loading: true
      };
    case profileConstants.PAY_CHARITY_REQUEST:
      return {
        ...state,
        txHash: '',
        loadingPay: true
      };
    case profileConstants.CONNECT_SUCCESS:
      return {
        ...state,
        address: action.res.address,
        phone: action.res.phoneNumber,
        daoBalance: action.res.daoBalance,
        userBalance: action.res.userBalance,
        txHash: '',
        loading: false
      };
    case profileConstants.PAY_CHARITY_SUCCESS:
      return {
        ...state,
        txHash: action.res.transactionHash,
        loadingPay: false
      };
    case profileConstants.CONTRIBUTION_SUCCESS:
      return {
        ...state,
        daoBalance: action.res.daoBalance,
        userBalance: action.res.userBalance,
        txHash: action.res.transactionHash,
        loading: false
      };
    case profileConstants.ROLE_CHECK_SUCCESS:
      return {
        ...state,
        isStakeholder: action.role.isStakeholder,
        isContributor: action.role.isContributor,
        contributed: action.role.contributed,
        loading: false
      };
    case profileConstants.ROLE_SUCCESS:
      return {
        ...state,
        txHash: action.res.transactionHash,
        loading: false,
      };
    case profileConstants.GET_VOTES_SUCCESS:
      return {
        ...state,
        votes: action.votes,
        loading: false,
      };
    case profileConstants.CONNECT_FAILED:
    case profileConstants.CONTRIBUTION_FAILED:
    case profileConstants.ROLE_CHECK_FAILED:
    case profileConstants.ROLE_FAILED:
    case profileConstants.GET_VOTES_FAILED:
      return {
        ...state,
        loading: false
      };
    case profileConstants.PAY_CHARITY_FAILED:
      return {
        ...state,
        loadingPay: false
      };
    default:
      return state
  }
}
```

## Proposal Reducer

```
import { proposalConstants } from '../../constants/proposalConstants';

const initialState = {
  proposals: [],
  proposal: {},
  txHash: '',
  loadingAll: false,
  loadingOne: false,
  loadingNew: false,
  loadingVote: false
};

export function proposal(state = initialState, action) {
  switch (action.type) {
    case proposalConstants.CREATE_PROPOSAL_REQUEST:
      return {
        ...state,
        loadingNew: true
      };
    case proposalConstants.GET_ALL_PROPOSAL_REQUEST:
      return {
        ...state,
        loadingAll: true
      };
    case proposalConstants.GET_PROPOSAL_REQUEST:
      return {
        ...state,
        loadingOne: true
      };
    case proposalConstants.VOTE_PROPOSAL_REQUEST:
      return {
        ...state,
        loadingVote: true
      };
    case proposalConstants.CREATE_PROPOSAL_SUCCESS:
      return {
        ...state,
        txHash: action.res.transactionHash,
        loadingNew: false
      };
    case proposalConstants.GET_ALL_PROPOSAL_SUCCESS:
      return {
        ...state,
        proposals: action.proposals,
        loadingAll: false,
      };
    case proposalConstants.GET_PROPOSAL_SUCCESS:
      return {
        ...state,
        proposal: action.proposal,
        loadingOne: false
      };
    case proposalConstants.VOTE_PROPOSAL_SUCCESS:
      return {
        ...state,
        txHash: action.res.transactionHash,
        loadingVote: false
      };
    case proposalConstants.CREATE_PROPOSAL_FAILED:
    case proposalConstants.GET_ALL_PROPOSAL_FAILED:
    case proposalConstants.GET_PROPOSAL_FAILED:
    case proposalConstants.VOTE_PROPOSAL_FAILED:
      return {
        ...state,
        loadingNew: false,
        loadingAll: false,
        loadingOne: false,
        loadingVote: false
      };
    default:
      return state;
  }
}
```

# The App State:

## Connect Wallet
```
import * as React from 'react';
import { SafeAreaView, StyleSheet, View } from 'react-native';
import { useDispatch, useSelector } from 'react-redux';
import { Layout, Button, Text, Spinner } from '@ui-kitten/components';

import { kit } from '../root';
import { profileActions } from '../store/actions';


export const WelcomePage = ({navigation}) => {
  const dispatch = useDispatch();
  const profile = useSelector(state => state.profile);

  const login = async () => {
    if (!kit.defaultAccount) {
      await dispatch(profileActions.connect());

      await dispatch(profileActions.getRole());

      navigation.navigate("ProposalsPage");

    } else navigation.navigate("ProposalsPage");
  }

  return (
    <Layout style={styles.container}>
      <Button
        raised='true'
        onPress={login}
        accessoryLeft={profile.loading ? loadingIndicator : ""}
      >Connect To Wallet</Button>

      <Text style={styles.text}>
        Welcome to the Charity example DAO
      </Text>
    </Layout>
  );
};

const loadingIndicator = (props) => {
  return(
    <View style={[props.style, styles.indicator]}>
      <Spinner size='small' status='basic' />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 20,
  },
  text: {
    paddingTop: 10,
  },
  indicator: {
    justifyContent: 'center',
    alignItems: 'center',
  },
});
```

## All Proposals Page
```
import * as React from 'react';
import { View, SafeAreaView, ScrollView, StyleSheet } from 'react-native';
import { Layout, Button, Text, TopNavigation, Divider, Icon, Card, useTheme, Modal, Input, List, Spinner } from '@ui-kitten/components';
import { useDispatch, useSelector } from 'react-redux';

import { CreateProposalModal, ViewProposalModal } from '../components';
import { profileActions, proposalActions } from '../store/actions';
import { kit } from '../root';

export const ProposalsPage = ({ navigation }) => {
  const [createVisible, setCreateVisible] = React.useState(false);
  const [viewVisible, setViewVisible] = React.useState(false);
  const [proposals, setProposals] = React.useState([])

  const dispatch = useDispatch();
  const store = useSelector(state => state.proposal);
  const profile = useSelector(state => state.profile);

  const theme = useTheme();

  React.useEffect(() => {
    if (!kit.defaultAccount) {
      navigation.navigate("WelcomePage");
    }
  });

  React.useEffect(() => {
    dispatch(proposalActions.getAllProposals());
    dispatch(profileActions.getVotes());
  }, []);

  React.useEffect(() => {
    getProposals();
  }, [store.proposals.length, profile.votes.length]);

  const getProposals = () => {
    const today = new Date().getTime() / 1000;
    const allProposals = store.proposals.filter(
      (element) => element[7] == false && element[2] > today
    );

    const filteredProposals = allProposals.filter(
      function(e) {
        return this.indexOf(e[0]) < 0;
      },
      profile.votes
    );

    setProposals(filteredProposals);
  };

  const getProposal = (id) => {
    dispatch(proposalActions.getProposal(id));

    setViewVisible(true);
  };

  const chaFooter = (props, info) => {
    return(
      <View {...props} style={{...props.style, flexDirection: 'row', justifyContent: 'space-evenly', padding: 8}}>
        <Text style={{color: theme['color-info-default']}}>For: {info.for}</Text>
        <Text style={{color: theme['color-danger-default']}}>Against: {info.against}</Text>
      </View>
    );
  };

  const cardItem = (info) => {
    return(
      <Card
        style={{borderColor: theme['color-primary-default'], marginVertical: 4}}
        footer={props => chaFooter(props, {for: info.item[3], against: info.item[4]})}
        onPress={() => getProposal(info.item[0])}>
        
        <Text category='s2' numberOfLines={4} ellipsizeMode='tail'>{info.item[5]}</Text>
      </Card>
    );
  };

  return (
    <SafeAreaView style={{ flex: 1 }}>
      <Layout style={{ flex: 1, alignItems: 'center', padding: 16 }}>

        {
          profile.isStakeholder ?
          <Button
            size='medium'
            style={{marginVertical: 8}}
            onPress={() => setCreateVisible(true)}
          >Create Proposal</Button> : null
        }

        {
          store.loadingAll ? <Spinner status='primary' size='giant' /> : 
          store.proposals.length !== 0 ?
          <List
          style={{backgroundColor: theme['color-basic-800']}}
          contentContainerStyle={{paddingHorizontal: 8, paddingVertical: 4}}
          data={proposals}
          renderItem={cardItem}/> : <Text>The list of proposal is empty</Text>
        }

        <CreateProposalModal
          setVisible={setCreateVisible}
          visible={createVisible}/>

        <ViewProposalModal
          setVisible={setViewVisible}
          visible={viewVisible}
          isProfile='false'/>

      </Layout>
    </SafeAreaView>
  );
};

const loadingIndicator = (props) => {
  return(
    <View style={{...props.style, justifyContent: 'center', alignItems: 'center'}}>
      <Spinner size='small' status='basic' />
    </View>
  );
}
```

## Profile Screen
```
import * as React from 'react';
import { View, SafeAreaView } from 'react-native';
import { Button, Card, Input, Layout, List, useTheme, Text, Spinner, ViewPager, Icon } from '@ui-kitten/components';
import { useDispatch, useSelector } from 'react-redux';

import { ViewProposalModal } from '../components';
import { profileActions, proposalActions } from '../store/actions';
import { contractInstance, kit } from '../root';

export const ProfilePage = ({ navigation }) => {
  const [visible, setVisible] = React.useState(false);
  const [paidVotes, setPaidVotes] = React.useState([]);
  const [payableVotes, setPayableVotes] = React.useState([]);
  const [inVotingVotes, setInVotingVotes] = React.useState([]);
  const [selectedIndex, setSelectedIndex] = React.useState(0);

  const dispatch = useDispatch();
  const profile = useSelector(state => state.profile);
  const store = useSelector(state => state.proposal);

  const amountInput = useInputState('Amount');

  const theme = useTheme();

  const contribute = async () => {
    const contributed = amountInput.value;
    if (contributed === '') {
      return;
    }

    await dispatch(profileActions.contribute(contributed));

    await dispatch(profileActions.grantRole(contributed));

    await dispatch(profileActions.getRole());

    amountInput.setValue('');
  };

  React.useEffect(() => {
    dispatch(profileActions.getVotes());
  }, []);

  React.useEffect(() => {
    getVotes();
  }, [profile.votes.length]);

  const getVotes = () => {
    const today = new Date().getTime() / 1000;
    const stakeholderVotes = store.proposals.filter(
      function(e) {
        return this.indexOf(e[0]) >= 0;
      },
      profile.votes
    );

    const stakePaidVotes = stakeholderVotes.filter(
      (element) => element[7] == true
    );

    const stakePayable = stakeholderVotes.filter(
      (element) => (element[2] <= today && element[7] == false)
    );

    const stakeInVoting = stakeholderVotes.filter(
      (element) => (element[2] > today && element[7] == false)
    );
    
    setPaidVotes(stakePaidVotes);
    setPayableVotes(stakePayable);
    setInVotingVotes(stakeInVoting);
  };

  const getProposal = (id) => {
    if (store.proposal.id == id) {
      setVisible(true);

      return;
    }

    dispatch(proposalActions.getProposal(id));

    setVisible(true);
  };

  const shouldLoadComponent = (index) => index == selectedIndex;
    
  const handleRightClick = () => {
    if (selectedIndex < 2) {
      setSelectedIndex(selectedIndex + 1);
    }
  };
  const handleLeftClick = () => {
    if (selectedIndex > 0) {
      setSelectedIndex(selectedIndex - 1);
    }
  };

  const cardItem = (info) => {
    return(
      <Card
        style={{borderColor: theme['color-primary-default'], marginVertical: 4}}
        footer={props => cardFooter(props, {for: info.item[3], against: info.item[4]})}
        onPress={() => getProposal(info.item[0])}>
        
        <Text category='s2' numberOfLines={4} ellipsizeMode='tail'>{info.item[5]}</Text>
      </Card>
    );
  };
  
  const cardFooter = (props, info) => {
    return(
      <View {...props} style={{...props.style, flexDirection: 'row', justifyContent: 'space-evenly', padding: 8}}>
        <Text style={{color: theme['color-info-default']}}>For: {info.for}</Text>
        <Text style={{color: theme['color-danger-default']}}>Against: {info.against}</Text>
      </View>
    );
  };

  const page = () => {
    return(
      <>
        <Layout style={{flexDirection: 'row', justifyContent: 'space-around'}}>
          <Button
            appearance='ghost'
            status='info'
            accessoryLeft={leftIcon}
            onPress={handleLeftClick}
            />
          <Text style={{alignSelf: 'center', marginVertical: 16}}>{selectedIndex == 0 ? 'In Voting' : selectedIndex == 1 ? 'Payable Proposals' : 'Paid Proposals'}</Text>
          <Button
            appearance='ghost'
            status='info'
            accessoryRight={rightIcon}
            onPress={handleRightClick}
            />
        </Layout>
        {
          (profile.loading || !profile.isStakeholder) ?
          <Text style={{backgroundColor: theme['color-basic-800']}}>{selectedIndex == 0 ? 'The list of "in voting" proposal is empty' : selectedIndex == 1 ? 'The list of "payable" proposal is empty' : 'The list of "paid" proposal is empty'}</Text> :
          <List
          style={{backgroundColor: theme['color-basic-800']}}
          contentContainerStyle={{paddingHorizontal: 8, paddingVertical: 4}}
          data={selectedIndex == 0 ? inVotingVotes : selectedIndex == 1 ? payableVotes : paidVotes}
          renderItem={cardItem}/>
        }
      </>
    );
  };

  const rightIcon = (props) => {
    if (selectedIndex != 2) {
      return(
        <Icon
          {...props}
          name='arrowhead-right-outline'/>
      );
    }

    return(null);
  };

  const leftIcon = (props) => {
    if (selectedIndex != 0) {
      return(
        <Icon
          {...props}
          name='arrowhead-left-outline'/>
      );
    }
    
    return(null);
  };

  return (
    <SafeAreaView style={{ flex: 1 }}>
      <Layout style={{ flex: 1, padding: 16 }}>
        <Card style={{flexDirection: 'column', justifyContent: 'space-between', borderColor: theme['color-basic-800']}}>
          <Input
            style={{marginVertical: 8}}
            size='medium'
            status='primary'
            keyboardType='numeric'
            placeholder='Enter amount to contribute'
            {...amountInput}/>
          
          <Button
            size='small'
            onPress={contribute}
            accessoryLeft={profile.loading ? loadingIndicator : ''}
            >
            CONTRIBUTE
          </Button>
        </Card>

        <ViewPager style={{height: 480}}
          selectedIndex={selectedIndex}
          shouldLoadComponent={shouldLoadComponent}
          onSelect={setSelectedIndex}>

            <Layout level='2'>
              {page()}
            </Layout>

            <Layout level='2'>
              {page()}
            </Layout>

            <Layout level='2'>
              {page()}
            </Layout>
          </ViewPager>

        <ViewProposalModal
          setVisible={setVisible}
          visible={visible}
          isPofile='true'/>
      </Layout>
    </SafeAreaView>
  );
};

const useInputState = (name, initialValue = '') => {
  const [value, setValue] = React.useState(initialValue);

  let caption;
  if (value === '') {
    caption = `${name} must not be empty`;
  }

  return {value, onChangeText: setValue, setValue, caption};
};

const loadingIndicator = (props) => {
  return(
    <View style={{...props.style, justifyContent: 'center', alignItems: 'center'}}>
      <Spinner size='small' status='basic' />
    </View>
  );
}
```

# Conclusion
In this tutorial, you learned how to build a complete Charity DAO. You wrote the complete smart contract code using the Solidity programming language. Built the App using React Native and then used Redux to tie the actions together!
The Smart Contract code is deployed on the Alfajores testnet. The contract allows the members to contribute to the DAO. 

We hope this was not the longest tutorial for you to follow. Lol. Congratulations if you have made it this far! There’s no limits to what you can build given the knowledge that you have thus acquired reading and following this tutorial! Go forth and build great things! 

# About the Authors
This tutorial was created by [Segun Ogundipe](https://www.linkedin.com/in/segun-ogundipe/) and [Emmanuel Oaikhenan](https://github.com/emmaodia).

