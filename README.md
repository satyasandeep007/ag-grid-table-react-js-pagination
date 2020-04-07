# ag-grid-table-react-js-pagination



import React, { Component, Fragment } from "react";
import axios from "axios";
import Breadcrumb from "../../../common/breadcrumb.component";
import PropTypes from "prop-types";
import { Table, Button, FormGroup, Input } from "reactstrap";
import { Link } from "react-router-dom";
import { CSVLink } from "react-csv";
import { connect } from "react-redux";
import { Redirect } from "react-router-dom";
import Alert from "../../../common/Alert";
import Config from "../../../../config";
import { AgGridReact } from 'ag-grid-react';
import Pagination from "react-js-pagination";
// require("bootstrap/less/bootstrap.less");

class NoticeAdd extends Component {
  constructor(props) {
    super(props);
    this.state = {
      link: "",
      modal: false,
      locations: [],
      description: "",
      message: "",
      style: {},
      unique_locationids: [],
      unique_vendors: [],
      anotherModal: false,
      Locationid: "",
      VendorName: "",
      Contact2: "",
      Contact3: "",
      page: 1,
      showText: false,
      columnDefs: [
        {
          headerName: "ID", field: "ID", sortable: true, filter: true
        },
        {
          headerName: "Vendor Name", field: "VendorName", sortable: true, filter: true
        },
        {
          headerName: "Mobile", field: "Mobile", sortable: true, filter: true
        },
        {
          headerName: "Contact2", field: "Contact2", sortable: true, filter: true, editable: true,
        },
        {
          headerName: "Contact3", field: "Contact3", sortable: true, filter: true, editable: true,
        },
        {
          headerName: "Edit", field: "Edit", editable: true,
          cellRendererFramework: (params) => {
            return <Link to={"/show-details/" + params.node.data.ID}>
              <Button
                color="danger"
                onClick={() =>
                  this.setState({
                    modal: !this.state.modal
                  })
                }
              >
                Change
        </Button>
            </Link>
          },
        }
      ],
      rowData: [],
      activePage: 1,
      itemsCountPerPage: 25,
      totalItemsCount: 200,
      pageRangeDisplayed: 5
    };
  }

  async componentDidMount() {
    let that = this;

    // get all the unique dates
    axios.get(`${Config.hostName}/unique`)
      .then((res) => {
        console.log(res.data);

        // save unique dates in component state object
        that.setState({ unique_locationids: res.data })
      })
      .catch((error) => {
        console.log(error)
      });
  }

  handlePageChange = async (pageNumber) => {
    console.log(`active page is ${pageNumber}`);
    await this.setState({ activePage: pageNumber });
    await this.componentWillMount();
  }

  componentWillMount = async () => {
    let { activePage } = this.state;

    var result = await axios.get(`${Config.hostName}/get/details/${activePage}`);
    console.log(result.data);

    await this.setState({
      locations: result.data,
      style: { display: 'none' }
    });
    await this.setState({
      rowData: this.state.locations.map(eachItem => {
        return {
          ID: eachItem.VendorID,
          VendorName: eachItem.TransportName,
          Mobile: eachItem.Mobile,
          Contact2: eachItem.Contact2,
          Contact3: eachItem.Contact3
        }
      })
    });
  }

  handleSearchShipment = async () => {
    try {
      let result = await axios.post(`${Config.hostName}/search`, {
        Locationid: this.state.Locationid,
        VendorName: this.state.VendorName
      });
      await this.setState({
        locations: result.data
      });

      await this.setState({
        rowData: this.state.locations.map(eachItem => {
          return {
            ID: eachItem.VendorID,
            VendorName: eachItem.TransportName,
            Mobile: eachItem.Mobile,
            Contact2: eachItem.Contact2,
            Contact3: eachItem.Contact3
          }
        })
      });
    } catch (e) {
      console.log(e);

    }

  }

  change = event => {
    this.setState({ unique_vendors: [] });
    event.preventDefault();
    this.setState({ Locationid: event.target.value });
    axios
      .post(`${Config.hostName}/get`, {
        Locationid: event.target.value
      })
      .then(res => {
        console.log(res.data);
        this.setState({
          unique_vendors: res.data
        });
      })
      .catch(err => {
        console.log(err);
      });
  };

  render() {
    let style = this.state.style;

    const defaultColDef = {
      flex: 1,
      minWidth: 90,
      resizable: true
    }

    if (
      this.props.auth.isAuthenticated &&
      this.props.auth.user.userType === "user"
    )
      return <Redirect to="/dashboard" />;
    else if (
      this.props.auth.isAuthenticated &&
      this.props.auth.user.userType === "staff"
    )
      return <Redirect to="/staff/notice" />;
    else if (!this.props.auth.isAuthenticated)
      return <Redirect to="/user/login" />;


    return (
      <div>
        <div className='loader-wrapper' style={style}>
          <div className="loader bg-white">
            <div className="line"></div>
            <div className="line"></div>
            <div className="line"></div>
            <div className="line"></div>
            <h4>Please Wait while its loading <span>&#x263A;</span></h4>
          </div>
        </div>

        {/*Container-fluid starts*/}
        <Breadcrumb link="ImportantDates" parent="Admin" />
        {/*Container-fluid Ends*/}

        {/*Container-fluid starts*/}
        <div className="container-fluid">
          <div className="edit-profile">
            <div className="row ">
              <div className="col-xl-12">
                <Alert />

                <div className="card-header">
                  <div className="row">
                    <div className="col-sm-12">
                      <h4 className="card-link mb-0">Vendor Details</h4>
                      <br />
                    </div>

                    <div className="col-sm-3">
                      <FormGroup>
                        <Input
                          type="select"
                          name="locationid"
                          value={this.state.Locationid}
                          id="locationid"
                          onChange={this.change}
                        >
                          <option value={""} disabled selected >Select Location</option>
                          {this.state.unique_locationids.map(locationid => (
                            <option value={locationid.Locationid}>
                              {
                                locationid.Locationid === 1 ? "Hyderabad" :
                                  locationid.Locationid === 2 ? "Bangalore"
                                    : "Chennai"
                              }
                            </option>
                          ))}
                        </Input>
                      </FormGroup>
                    </div>

                    <div className="col-sm-3">
                      <FormGroup>
                        <Input
                          type="select"
                          name="VendorName"
                          value={this.state.VendorName}
                          id="VendorName"
                          onChange={event => {
                            this.setState({ VendorName: event.target.value });
                          }}
                        >
                          <option value={""} disabled selected  >Select Vendor</option>

                          {this.state.unique_vendors.map(vendor => (
                            <option value={vendor.TransportName}>
                              {vendor.TransportName}
                            </option>
                          ))}
                        </Input>
                      </FormGroup>
                    </div>

                    <div className="col-sm-3">
                      <button
                        // type="submit"
                        className="btn btn-primary"
                        onClick={this.handleSearchShipment}
                      >
                        Search Vendor
                      </button>
                    </div>

                    <div className="col-sm-3">
                      <Link to={"/add-new"}>
                        <button
                          type="submit"
                          onClick={() =>
                            this.setState({
                              anotherModal: !this.state.anotherModal
                            })
                          }
                          className="btn btn-primary"
                        >
                          Add New
                        </button>
                      </Link>
                    </div>

                  </div>

                  <div className="card-options">
                    <a
                      href="/#"
                      className="card-options-Fade"
                      data-toggle="card-Fade"
                    >
                      <i className="fe fe-chevron-up"></i>
                    </a>
                    <a
                      href="/#"
                      className="card-options-remove"
                      data-toggle="card-remove"
                    >
                      <i className="fe fe-x"></i>
                    </a>
                  </div>
                </div>

                <div className="card-body">
                  <div
                    className="ag-theme-balham"
                    style={{
                      height: '500px',
                      width: '100%'
                    }}
                  >
                    <AgGridReact
                      columnDefs={this.state.columnDefs}
                      rowData={this.state.rowData}
                      defaultColDef={defaultColDef}
                      onGridReady={this.onGridReady}
                    >
                    </AgGridReact>
                  </div>
                  <div style={{ margin: "5vh auto", marginLeft: "25%", justifyContent: "center" }}>
                    <Pagination
                      activePage={this.state.activePage}
                      itemsCountPerPage={this.state.itemsCountPerPage}
                      totalItemsCount={this.state.totalItemsCount}
                      pageRangeDisplayed={this.state.pageRangeDisplayed}
                      onChange={this.handlePageChange.bind(this)}
                      prevPageText='prev'
                      nextPageText='next'
                      firstPageText='first'
                      lastPageText='last'
                      itemClass="page-item"
                      linkClass="page-link"
                    />
                  </div>
                </div>

              </div>
            </div>
          </div>
        </div>
      </div>
    );
  }


}

NoticeAdd.propTypes = {
  auth: PropTypes.object.isRequired,
  setAlert: PropTypes.func.isRequired
};

const mapStateToProps = state => ({
  auth: state.auth
});

export default connect(mapStateToProps, { setAlert })(NoticeAdd);
